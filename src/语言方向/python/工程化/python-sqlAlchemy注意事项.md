- [关于建表](#关于建表)
  - [不要混用Orm和Core](#不要混用orm和core)
    - [1. 关系表混用](#1-关系表混用)
    - [2. 基础ORM表](#2-基础orm表)
  - [关于字段顺序](#关于字段顺序)
- [插件](#插件)
  - [监听事件](#监听事件)
    - [1. **核心（Core）事件**](#1-核心core事件)
    - [2. **ORM 事件**](#2-orm-事件)
    - [3. **其他事件**](#3-其他事件)
  - [总结](#总结)
  - [软删除](#软删除)
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

## 插件


### 监听事件
> https://docs.sqlalchemy.org/en/20/orm/events.html#sqlalchemy.orm.SessionEvents.do_orm_execute


在 SQLAlchemy 2.0 中，事件监听器（event listeners）的体系非常丰富，涵盖了 ORM、Core、引擎（Engine）、连接（Connection）等多个层面。以下是主要的事件类别及常见监听器类型：

#### 1. **核心（Core）事件**
主要针对引擎、连接、SQL 执行等底层操作：
- **`Engine` 相关**：`connect`（连接创建时）、`begin`（事务开始）、`commit`（事务提交）、`rollback`（事务回滚）、`close`（连接关闭）等。
- **`Connection` 相关**：`execute`（执行 SQL 前）、`after_execute`（执行 SQL 后）、`begin_nested`（嵌套事务开始）等。
- **DDL 相关**：`before_create`（创建表前）、`after_create`（创建表后）、`before_drop`（删除表前）、`after_drop`（删除表后）等，用于监听数据表结构变更。


#### 2. **ORM 事件**
针对会话（Session）、模型（Model）、关系（Relationship）等 ORM 组件：
- **`Session` 相关**：`do_orm_execute`（ORM 操作执行时，如查询、新增等）、`before_flush`（提交前刷新）、`after_flush`（提交后刷新）、`before_commit`（提交前）、`after_commit`（提交后）等。
- **模型实例相关**：`init`（实例初始化时）、`load`（实例从数据库加载时）、`before_insert`（插入前）、`after_insert`（插入后）、`before_update`（更新前）、`after_update`（更新后）、`before_delete`（删除前）、`after_delete`（删除后）等。
- **关系（Relationship）相关**：`append`（向关系集合添加元素时）、`remove`（从关系集合移除元素时）、`set`（设置关系值时）等。


#### 3. **其他事件**
- **`Mapper` 相关**：`after_configured`（映射器配置完成后），用于全局映射器初始化后的操作。
- **`Query` 相关**：在 SQLAlchemy 1.x 中有 `before_compile` 等，但 2.0 中部分功能已整合到 `Session` 事件中。
- **池（Pool）相关**：`checkout`（从连接池检出连接时）、`checkin`（连接归还池时）、`connect`（池创建新连接时）等，用于连接池管理。


### 总结
SQLAlchemy 2.0 的事件监听器没有绝对固定的“数量”，因为每个类别下还有细分的触发点和参数，但核心常用的事件类型约有 **30+**。具体可参考官方文档的 [事件参考列表](https://docs.sqlalchemy.org/en/20/core/events.html)，根据实际需求选择合适的监听器。

事件机制的灵活性使得可以在 SQLAlchemy 的各个生命周期插入自定义逻辑（如软删除、数据校验、日志记录等）。


### 软删除

1. 模型
```python
class LogicMixin(MappedAsDataclass):
    """逻辑 Mixin 数据类"""

    id: Mapped[snowflake_id_key] = mapped_column(init=False)

    deleted_at: Mapped[datetime | None] = mapped_column(
        DateTime(timezone=True),
        init=False,
        default=None,
        sort_order=997,
        comment="删除时间，为空则未删除",
    )

    def soft_delete(self):
        """逻辑删除当前记录"""
        self.deleted_at = timezone.now()

```
2. 监听器
```python

def setup_soft_delete_plug() -> None:
    # 注册事件监听：拦截查询并添加过滤条件
    @event.listens_for(Session, "do_orm_execute")
    def _add_filtering_deleted_at(execute_state):
        """
        自动过滤掉被软删除的数据。
        deleted_at不为null即为被软删除。
        使用以下方法可以获得被软删除的数据。
        select(...).execution_options(include_deleted=True)
        """
        if (
            execute_state.is_select
            and not execute_state.is_column_load
            and not execute_state.is_relationship_load
            and not execute_state.execution_options.get("include_deleted", False)
        ):
            execute_state.statement = execute_state.statement.options(
                with_loader_criteria(
                    LogicMixin,
                    lambda cls: cls.deleted_at.is_(
                        None
                    ),  # deleted_at列为空则为未被软删除
                    include_aliases=True,
                )
            )

```
3. 使用
```python
query = (
    select(WjChatSession.id, WjChatSession.name) 
    .execution_options(include_deleted=True) # 一定要放到select()方法之后才能生效
    .filter(WjChatSession.user_id == user_id)
)
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
