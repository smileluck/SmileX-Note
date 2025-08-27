

# 创建一个web项目
最近在公司从零搭建一个物联网架构项目，打算把相关资料和文档记录一下

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

## 安装步骤
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
