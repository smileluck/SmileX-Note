- [fastapi-命名规则.md](#fastapi-命名规则md)
  - [一、项目结构命名](#一项目结构命名)
  - [二、Python 基础命名（遵循 PEP8）](#二python-基础命名遵循-pep8)
  - [三、FastAPI 特有组件命名](#三fastapi-特有组件命名)
    - [1. 路由（Router）](#1-路由router)
    - [2. 模型（Pydantic/ORM）](#2-模型pydanticorm)
    - [3. 依赖项（Dependency）](#3-依赖项dependency)
    - [4. 路径与标签](#4-路径与标签)
    - [5. 异常与响应](#5-异常与响应)
  - [四、API 接口命名（RESTful 风格）](#四api-接口命名restful-风格)
  - [五、配置与环境变量](#五配置与环境变量)
  - [总结](#总结)


## fastapi-命名规则.md
在 FastAPI 项目中，良好的命名规范有助于提高代码可读性、可维护性和团队协作效率。以下是结合 FastAPI 特性和 Python 社区最佳实践的命名规则建议：


### 一、项目结构命名
FastAPI 项目通常采用模块化结构，目录和文件命名需清晰反映功能：

| 元素         | 命名规则                          | 示例                            |
| ------------ | --------------------------------- | ------------------------------- |
| 项目根目录   | 小写字母 + 下划线（`snake_case`） | `fastapi_blog`、`user_service`  |
| 核心模块目录 | 小写字母 + 下划线                 | `app`、`api`、`core`、`models`  |
| 子模块目录   | 小写字母 + 下划线                 | `app/api/v1`、`app/db`          |
| Python 文件  | 小写字母 + 下划线                 | `user_router.py`、`database.py` |
| 配置文件     | 小写字母 + 下划线                 | `settings.py`、`config.yaml`    |


### 二、Python 基础命名（遵循 PEP8）
FastAPI 基于 Python，需遵守 PEP8 规范：

| 元素       | 命名规则                          | 示例                           |
| ---------- | --------------------------------- | ------------------------------ |
| 变量       | 小写字母 + 下划线（`snake_case`） | `user_id`、`page_size`         |
| 函数       | 小写字母 + 下划线                 | `get_user()`、`create_order()` |
| 类         | 首字母大写（`PascalCase`）        | `UserModel`、`AuthService`     |
| 常量       | 全大写 + 下划线                   | `MAX_RETRY`、`API_PREFIX`      |
| 模块级变量 | 小写字母 + 下划线                 | `db_session`、`redis_client`   |


### 三、FastAPI 特有组件命名

#### 1. 路由（Router）
- 路由文件名：`{资源}_router.py`（体现负责的资源）  
  示例：`user_router.py`、`order_router.py`
- 路由实例：`{资源}_router`  
  示例：
  ```python
  # user_router.py
  from fastapi import APIRouter
  user_router = APIRouter(prefix="/users", tags=["用户管理"])
  ```
- 路由函数：`{动作}_{资源}` 或 `{资源}_{动作}`（动词在前更清晰）  
  示例：
  ```python
  @user_router.get("/{user_id}")
  def get_user(user_id: str):  # 获取用户
      ...

  @user_router.post("")
  def create_user(user: UserCreate):  # 创建用户
      ...
  ```


#### 2. 模型（Pydantic/ORM）
- Pydantic 模型（数据验证/序列化）：  
  - 输入模型（请求体）：`{资源}Create` 或 `{资源}Update`  
  - 输出模型（响应体）：`{资源}Out` 或直接用 `{资源}`  
  - 示例：
    ```python
    from pydantic import BaseModel

    class UserCreate(BaseModel):  # 创建用户的请求模型
        username: str
        email: str

    class UserOut(BaseModel):  # 查询用户的响应模型
        id: str
        username: str
    ```

- ORM 模型（数据库映射）：  
  - 类名：`{资源}Model`（与数据库表对应）  
  - 示例：
    ```python
    from sqlalchemy.ext.declarative import declarative_base

    Base = declarative_base()

    class UserModel(Base):  # 用户表模型
        __tablename__ = "users"
        id: str
        username: str
    ```


#### 3. 依赖项（Dependency）
- 依赖函数：`get_{资源}` 或 `{功能}_dependency`  
  示例：
  ```python
  def get_db():  # 获取数据库会话的依赖
      db = SessionLocal()
      try:
          yield db
      finally:
          db.close()

  def auth_dependency(token: str = Header()):  # 认证依赖
      ...
  ```


#### 4. 路径与标签
- 路径（Path）：使用小写字母 + 短横线（`kebab-case`）或下划线，建议与资源对应  
  示例：`/user-profiles`、`/order-items`
- 标签（Tags）：使用中文或 PascalCase 英文，用于 API 文档分组  
  示例：
  ```python
  @user_router.get("/", tags=["用户管理"])  # 中文标签（推荐，更直观）
  def list_users():
      ...

  @order_router.post("/", tags=["OrderOperations"])  # 英文标签
  def create_order():
      ...
  ```


#### 5. 异常与响应
- 自定义异常：`{错误类型}Exception`  
  示例：`ResourceNotFoundError`、`ValidationError`
- 响应状态码：使用 FastAPI 内置常量或直接写数字（建议用常量提高可读性）  
  示例：
  ```python
  from fastapi import status

  @user_router.post("", status_code=status.HTTP_201_CREATED)  # 推荐：使用常量
  def create_user():
      ...
  ```


### 四、API 接口命名（RESTful 风格）
遵循 RESTful 设计原则，通过 HTTP 方法和路径表达资源操作：

| 操作         | HTTP 方法 | 路径示例           | 路由函数示例           |
| ------------ | --------- | ------------------ | ---------------------- |
| 获取列表     | GET       | `/users`           | `get_users()`          |
| 获取单个资源 | GET       | `/users/{user_id}` | `get_user(user_id)`    |
| 创建资源     | POST      | `/users`           | `create_user(user)`    |
| 更新资源     | PUT/PATCH | `/users/{user_id}` | `update_user(user_id)` |
| 删除资源     | DELETE    | `/users/{user_id}` | `delete_user(user_id)` |


### 五、配置与环境变量
- 配置类：`Settings` 或 `{模块}Settings`  
  示例：
  ```python
  from pydantic_settings import BaseSettings

  class Settings(BaseSettings):  # 全局配置
      api_prefix: str = "/api/v1"
      db_url: str = "postgresql://user:pass@localhost/db"
  ```
- 环境变量：全大写 + 下划线，前缀建议加项目名  
  示例：`APP_API_PREFIX`、`DB_CONN_TIMEOUT`


### 总结
FastAPI 命名的核心原则是：**清晰易懂、保持一致性、反映功能用途**。建议团队内部统一规范，并借助工具（如 `flake8`、`black`）自动检查格式，减少风格冲突。遵循这些规则能让代码更易维护，尤其在大型项目和团队协作中效果显著。