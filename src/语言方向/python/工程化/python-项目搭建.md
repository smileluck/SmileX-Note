- [创建一个web项目](#创建一个web项目)
  - [相关资料](#相关资料)
  - [技术栈](#技术栈)
  - [安装依赖](#安装依赖)
  - [通用工具](#通用工具)
    - [数据库](#数据库)


# 创建一个web项目
最近在公司从零搭建一个物联网架构项目，打算把相关资料和文档记录一下

## 相关资料
- [fastapi_best_architecture](https://github.com/fastapi-practices/fastapi_best_architecture/)
- [fastapi_best_architecture_docs](https://fastapi-practices.github.io/fastapi_best_architecture_docs/backend/reference/pagination.html)
- [fastapi](https://fastapi.tiangolo.com/zh/how-to/)
- [padantic](https://docs.pydantic.dev/latest/concepts/pydantic_settings/)
- [sqlalchemy](https://www.sqlalchemy.org/)

## 技术栈
- **python**: 3.11.13
- **框架**: FastAPI
- **数据库**: PostgreSQL
- **数据库驱动**: asyncpg 
- **缓存**: Redis
- **ORM**: SQLAlchemy 2.0
- **认证**: JWT (JSON Web Token) / fastapi-user
- **视频通信**: aiortc (WebRTC)
- **接口验证**: Pydantic
- **socket**: socket(python内置)
- **其它**: httpx

## 安装依赖
1. 安装fastapi和unicorn
   ```bash
   pip install fastapi "uvicorn[standard]"
   ```
2. orm sqlalchemy 2.x + asyncpg 驱动。使用异步，如果需要同步则使用 psycopg2
    ```bash
    pip install SQLAlchemy asyncpg

    # 使用 asyncpg 则 链接要修改这样
    # postgresql+asyncpg://user:pwd@localhost:5432/wujie_cloud
    # 使用 psycopg2 则 链接要修改这样
    # postgresql+psycopg2://user:pwd@localhost:5432/choice_db

    ```
3. fastapi-users
    ```bash
    pip install fastapi-users[sqlalchemy]
    ```

## 通用工具
### 数据库
```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker

# 数据库连接URL（asyncpg适配器）
DATABASE_URL = "postgresql+asyncpg://user:password@localhost:5432/your_db"

DATABASE_POOL = None
DATABASE_SESSION_MAKER = None

async def init_pool():
    """初始化数据库连接池"""
    global DATABASE_POOL,DATABASE_SESSION_MAKER
    # 创建异步引擎（包含连接池配置）
    DATABASE_POOL = await create_async_engine(
        DATABASE_URL,
        echo=False,  # 生产环境关闭SQL日志
        pool_size=10,  # 核心连接数：根据并发量调整（如10个worker，每个处理10并发，设为10）
        max_overflow=20,  # 最大溢出连接：突发流量时临时创建的连接数
        pool_recycle=300,  # 连接5分钟（300秒）后自动回收，避免PostgreSQL关闭闲置连接
        pool_pre_ping=True,  # 获取连接前检测有效性（防止连接失效）
        pool_use_lifo=True  # 优先使用最近创建的连接（提升性能）
    )

    # 异步会话工厂（从连接池获取连接）
    DATABASE_SESSION_MAKER = sessionmaker(
        bind=DATABASE_POOL,
        class_=AsyncSession,
        autoflush=False,
        autocommit=False,
        expire_on_commit=False
    )
    
async def close_pool():
    """关闭数据库连接池"""
    global DATABASE_POOL
    if DATABASE_POOL is not None:  # 确保连接池已初始化
        await DATABASE_POOL.dispose()   
        DATABASE_POOL = None
    if DATABASE_SESSION_MAKER is not None:  # 确保会话工厂已初始化
        DATABASE_SESSION_MAKER.close()
        DATABASE_SESSION_MAKER = None
        
async def get_conn():
    """获取数据库连接"""
    global DATABASE_SESSION_MAKER
    async with DATABASE_SESSION_MAKER() as session:
        try:
            yield session
        finally:
            await session.close()  # 会话关闭后，连接返回连接池（而非销毁）

```