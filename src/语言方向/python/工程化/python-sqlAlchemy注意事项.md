- [关于建表](#关于建表)
  - [不要混用Orm和Core](#不要混用orm和core)
    - [1. 关系表混用](#1-关系表混用)
    - [2. 基础ORM表](#2-基础orm表)
  - [关于字段顺序](#关于字段顺序)
- [异常记录](#异常记录)
  - [将sqlAlchemy的ORM对象转换为pydantic](#将sqlalchemy的orm对象转换为pydantic)


## 关于建表

### 不要混用Orm和Core

#### 1. 关系表混用
在 `Table` 中使用了`mapped_column`,应该使用`Column`字段
```python

# 角色菜单关联表(重复定义是为了确保每个文件独立可用)
sys_role_menu_association = Table(
    "sys_role_menu",
    Base.metadata,
    Column("role_id", ForeignKey("sys_role.id", ondelete="CASCADE"), primary_key=True),
    Column("menu_id", ForeignKey("sys_menu.id", ondelete="CASCADE"), primary_key=True),
    Column(
        "permission",
        String(255),
        nullable=False,
        default="read",
        comment="权限类型：read, write, delete等",
    ),
)
```

#### 2. 基础ORM表
创建基础表的字段，可以考虑使用 `sqlAlchemy.orm` 库的，减少常见的 `sql` 操作。

1. 使用 `mapped_column` 而不是 `Column`
```python

class SysMenu(Base):
    """
    系统菜单表
    存储系统菜单、目录和按钮等权限点
    """

    # 菜单层级结构
    parent_id: Mapped[int | None] = mapped_column(
        ForeignKey("sys_menu.id", ondelete="CASCADE"),
        nullable=True,
        comment="父菜单ID，顶级菜单为0或NULL",
    )

    # 菜单图标（没有默认值）
    icon: Mapped[str] = mapped_column(String(50), nullable=True, comment="菜单图标")

    # 菜单基本信息
    name: Mapped[str] = mapped_column(String(100), nullable=False, comment="菜单名称")

    path: Mapped[str] = mapped_column(String(255), nullable=True, comment="路由路径")

    component: Mapped[str] = mapped_column(
        String(255), nullable=True, comment="组件路径"
    )

    redirect: Mapped[str] = mapped_column(
        String(255), nullable=True, comment="重定向路径"
    )

    # 权限标识
    permission: Mapped[str] = mapped_column(
        String(100), nullable=True, comment="权限标识，如 sys:user:list"
    )

    # 路由元信息
    meta_title: Mapped[str] = mapped_column(
        String(100), nullable=True, comment="路由标题"
    )

    meta_icon: Mapped[str] = mapped_column(
        String(50), nullable=True, comment="路由图标"
    )
    
    # 关联关系
    # 与角色表的多对多关系
    roles: Mapped[List["SysRole"]] = relationship(
        secondary=sys_role_menu_association,
        back_populates="menus",
        lazy="selectin",
    )

    # 与自身的一对多关系（子菜单）
    children: Mapped[List["SysMenu"]] = relationship(
        back_populates="parent",
        lazy="selectin",
        cascade="all, delete-orphan",
    )

    # 与自身的多对一关系（父菜单）
    parent: Mapped["SysMenu"] = relationship(
        back_populates="children",
        remote_side="SysMenu.id",
        lazy="joined",
    )


    meta_hidden: Mapped[bool] = mapped_column(
        Boolean, default=False, comment="是否隐藏菜单"
    )

    meta_affix: Mapped[bool] = mapped_column(
        Boolean, default=False, comment="是否固定标签"
    )

    meta_breadcrumb: Mapped[bool] = mapped_column(
        Boolean, default=True, comment="是否显示面包屑"
    )

    # 状态信息
    status: Mapped[bool] = mapped_column(
        Boolean, default=True, comment="状态：True-启用，False-禁用"
    )

    # 菜单类型（有默认值）
    type: Mapped[MenuType] = mapped_column(
        Enum(MenuType),
        default=MenuType.MENU,
        comment="菜单类型：catalog-目录, menu-菜单, button-按钮, external-外部链接",
    )

    # 排序字段
    sort: Mapped[int] = mapped_column(default=0, comment="排序号")
```



### 关于字段顺序

务必按照顺序【无默认值-> 关系字段-> 有默认值字段】来定义。

```python


class SysRole(Base):
    """
    系统角色表
    存储角色信息及其关联的权限配置
    """

    # 角色基本信息
    name: Mapped[str] = mapped_column(
        String(100), unique=True, nullable=False, comment="角色名称"
    )

    code: Mapped[str] = mapped_column(
        String(100), unique=True, index=True, nullable=False, comment="角色编码"
    )

    description: Mapped[str] = mapped_column(Text, nullable=True, comment="角色描述")

    # 关联关系
    # 与用户表的多对多关系
    users: Mapped[List["SysUser"]] = relationship(
        secondary=sys_user_role_association,
        back_populates="roles",
        lazy="selectin",
    )

    # 与菜单表的多对多关系
    menus: Mapped[List["SysMenu"]] = relationship(
        secondary=sys_role_menu_association,
        back_populates="roles",
        lazy="selectin",
    )

    # 状态信息
    status: Mapped[bool] = mapped_column(
        Boolean, default=True, comment="状态：True-启用，False-禁用"
    )

    is_default: Mapped[bool] = mapped_column(
        Boolean, default=False, comment="是否为默认角色"
    )

    is_system: Mapped[bool] = mapped_column(
        Boolean, default=False, comment="是否为系统内置角色"
    )

    # 排序字段
    sort: Mapped[int] = mapped_column(default=0, comment="排序号")
```

## 异常记录

### 将sqlAlchemy的ORM对象转换为pydantic
- 错误信息：errors=[': Input should be a valid dictionary or instance of RobotResponse']
- 解决办法：需要在pydantic的BaseModel中添加一个配置
  ```python
    class ChatMessageResponse(BaseModel):
        """聊天消息响应模型"""
        model_config = ConfigDict(from_attributes=True)

        id: int
        session_id: int
        sender_type: str
        content: str
        content_type: str
        created_at: datetime
        voice_url: Optional[str] = None
        voice_duration: Optional[float] = None
        response_time: Optional[float] = None
  ```

- 错误信息：会没有匹配到字段，提示异常errors=['id: Field required', 'session_id: Field required', 'sender_type: Field required', 'content: Field required', 'content_type: Field required', 'created_at: Field required'], request_id=-
- 原因：这是因为sqlAlchemy查询的方式有问题，导致返回的信息存在多层[truple]
- 解决办法：
    ```python
    
        stmt = (
            select(WjChatMessage)
            .where(WjChatMessage.session_id == session_id)
            .order_by(WjChatMessage.created_at)
            .offset(offset)
            .limit(page_size)
        )
        result = await self.session.execute(stmt)
        messages = result.all() # 这种列表不行，返回格式不符合
        
        messages = result.scalars().all() # 这种即可成功通过

        return [ChatMessageResponse.model_validate(message) for message in messages]
    ```
