# 前言
记录于 2025年 09 月 11 日


1. 环境参考
   1. python 3.10.4
   2. fastapi 0.116.1
   3. fastapi-users 9.5.0
2. 问题
   1. redis策略，同步和异步冲突
   2. jwt策略，登陆成功后，token传入会提示401
   3. 默认基于email+password的，不是这一套的不要用

# 使用问题记录
异常问题记录：
1. 使用异步线程池，依赖Depends会出现set异常
2. write_token异常，加await会出现 object bool can't be used in 'await' expression。不加会提示 coroutine not str。
3. token传入后异常。


# 源码

1. redis
```python
# redis

from typing import Optional, Any
import redis.asyncio as redis
from core.config import settings
from core.log import logger


class RedisPool:
    """异步 Redis 连接池管理类"""

    _pool: Optional[redis.ConnectionPool] = None

    @classmethod
    async def init_pool(cls) -> None:
        """初始化连接池"""
        if not cls._pool:
            cls._pool = redis.ConnectionPool(
                host=settings.REDIS.HOST,
                port=settings.REDIS.PORT,
                db=settings.REDIS.DB,
                password=settings.REDIS.PASSWORD,
                decode_responses=settings.REDIS.DECODE_RESPONSES,
                max_connections=settings.REDIS.MAX_CONNECTIONS,
                socket_connect_timeout=settings.REDIS.SOCKET_CONNECT_TIMEOUT,
                socket_timeout=settings.REDIS.SOCKET_TIMEOUT,
                socket_keepalive=settings.REDIS.SOCKET_KEEPALIVE,
                socket_keepalive_options=settings.REDIS.SOCKET_KEEPALIVE_OPTIONS,
                retry_on_timeout=settings.REDIS.RETRY_ON_TIMEOUT,
            )

            # 验证连接池有效性
            async with redis.Redis(connection_pool=cls._pool) as client:
                try:
                    await client.ping()
                except Exception as e:
                    cls._pool = None
                    raise ConnectionError(f"Redis 连接池初始化失败: {str(e)}")

    @classmethod
    def get_client(cls) -> redis.Redis:
        """获取一个 Redis 连接客户端"""
        if cls._pool is None:
            raise RuntimeError("Redis 连接池尚未初始化，请先调用 init_pool()")
        return redis.Redis(connection_pool=cls._pool)

    @classmethod
    async def close_pool(cls) -> None:
        """关闭连接池"""
        if cls._pool is not None:
            await cls._pool.disconnect()
            cls._pool = None


# FastAPI 依赖项：获取一个 Redis 客户端（自动从连接池获取）
async def get_redis_client() -> redis.Redis:
    """获取Redis客户端实例，作为FastAPI依赖项"""
    return RedisPool.get_client()

```

2. user_manager
```python
from typing import Optional, Union, Dict, Any
from fastapi import Depends, Request
from fastapi_users.authentication import (
    AuthenticationBackend,
    BearerTransport,
    RedisStrategy,
)
from fastapi_users import BaseUserManager, IntegerIDMixin, exceptions, models, schemas
from fastapi_users_db_sqlalchemy import SQLAlchemyUserDatabase
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from core.config import settings
from app.models.sys.user import SysUser
from core.database import get_conn
from core.redis import get_redis_client, RedisPool, get_redis_util, RedisUtil
from redis.asyncio.client import Redis


class UserManager(IntegerIDMixin, BaseUserManager[SysUser, int]):
    """
    用户管理器类
    负责用户的创建、认证、密码重置等操作
    """

    # 重写 authenticate 方法以支持用户名认证
    async def authenticate(self, username: str, password: str) -> Optional[SysUser]:
        """
        验证用户凭据
        通过用户名查找用户并验证密码

        Args:
            username: 用户名
            password: 密码

        Returns:
            SysUser: 用户对象，如果验证成功；否则返回None
        """
        try:
            # 按用户名查询用户
            # 注意：SQLAlchemyUserDatabase 默认没有 get_by_username 方法
            # 这里我们使用自定义查询来通过用户名查找用户
            stmt = select(SysUser).where(SysUser.username == username)
            result = await self.user_db.session.execute(stmt)
            user = result.scalar_one_or_none()

            if user is None:
                # 用户不存在，模拟密码验证耗时，防止时序攻击
                await self.password_helper.hash(password)
                return None

            # 验证密码
            if self.password_helper.verify_and_update(password, user.password):
                return user
            return None
        except Exception as e:
            # 发生任何异常都返回None
            print(f"认证过程中发生异常: {e}")
            await self.password_helper.hash(password)
            return None

    async def validate_password(
        self, password: str, user: Union[schemas.UC, SysUser]
    ) -> None:
        """
        验证密码是否符合要求
        可以在这里添加密码强度检查逻辑

        Args:
            password: 用户输入的密码
            user: 用户对象或创建用户的模式

        Raises:
            exceptions.InvalidPasswordException: 当密码不符合要求时抛出
        """
        if len(password) < 8:
            raise exceptions.InvalidPasswordException(
                reason="密码长度必须至少为8个字符"
            )
        # 可以添加更多密码验证规则，如包含大小写字母、数字、特殊字符等

    async def on_after_register(self, user: SysUser, request: Optional[Request] = None):
        """
        用户注册后的回调
        可以在这里实现用户注册后的额外逻辑，如发送欢迎邮件等

        Args:
            user: 注册的用户对象
            request: 请求对象（可选）
        """
        print(f"用户 {user.id} 注册成功")
        # 这里可以添加注册成功后的逻辑，如发送邮件通知等

    async def on_after_login(
        self,
        user: SysUser,
        request: Optional[Request] = None,
        response: Optional[Any] = None,
    ):
        """
        用户登录后的回调
        可以在这里实现用户登录后的额外逻辑，如更新最后登录时间、记录登录IP等

        Args:
            user: 登录的用户对象
            request: 请求对象（可选）
            response: 响应对象（可选）
        """
        print(f"用户 {user.id} 登录成功")
        # 这里可以添加登录成功后的逻辑，如更新登录时间、记录登录IP等


async def get_user_db(session: AsyncSession = Depends(get_conn)):
    """
    获取用户数据库实例

    Args:
        session: 数据库会话

    Yields:
        SQLAlchemyUserDatabase: 用户数据库实例
    """
    yield SQLAlchemyUserDatabase(session, SysUser)


async def get_user_manager(user_db: SQLAlchemyUserDatabase = Depends(get_user_db)):
    """
    获取用户管理器实例

    Args:
        user_db: 用户数据库实例

    Yields:
        UserManager: 用户管理器实例
    """
    yield UserManager(user_db)


# 定义认证后端
bearer_transport = BearerTransport(tokenUrl="/admin/auth/login")


# 只显示需要修改的部分
def get_redis_strategy(
    redis_client: Redis = Depends(get_redis_client),
) -> RedisStrategy:
    """
    获取Redis策略实例
    使用Redis连接池和设置中的密钥和过期时间

    Returns:
        RedisStrategy: Redis策略实例
    """
    return RedisStrategy(
        redis_client, lifetime_seconds=3600, key_prefix="sys_user_token:")


# 创建认证后端实例
auth_backend = AuthenticationBackend(
    name="redis",
    transport=bearer_transport,
    get_strategy=get_redis_strategy,
)

```

3. auth_router.py
```python
from fastapi import APIRouter, Depends, Request, Response
from fastapi_users import BaseUserManager, FastAPIUsers
from pydantic import BaseModel
from redis import Redis

from app.models.sys.user import SysUser
from core.config import settings
from core.response import (
    ResponseModel,
    ResponseSchemaModel,
    response_base,
    CustomResponseCode,
    CustomErrorCode,
)
from .user_manager import get_user_manager, auth_backend, get_redis_strategy

from core.redis import get_redis_client, RedisPool, get_redis_util, RedisUtil


# 登录响应数据模型
class LoginResponseData(BaseModel):
    """登录接口返回的数据模型"""

    access_token: str
    token_type: str


# 注册响应数据模型
class RegisterResponseData(BaseModel):
    """注册接口返回的数据模型"""

    user_id: int


# 用户信息响应数据模型
class UserInfoResponseData(BaseModel):
    """用户信息接口返回的数据模型"""

    id: int
    username: str
    nickname: str
    email: str | None
    phone: str | None
    avatar: str | None
    is_superuser: bool
    status: bool
    last_login_at: str | None
    last_login_ip: str | None


# 创建FastAPIUsers实例
fastapi_users = FastAPIUsers[SysUser, int](
    get_user_manager,
    [auth_backend],
)

# 获取当前用户的依赖项
current_active_user = fastapi_users.current_user(active=True)
current_superuser = fastapi_users.current_user(active=True, superuser=True)


# 创建认证路由
router = APIRouter(prefix="/auth", tags=["认证"])


# 只显示需要修改的部分
@router.get("/temp", response_model=ResponseModel)
async def temp(redis_client: Redis = Depends(get_redis_client)):
    await redis_client.set("test", "456")
    value = await redis_client.get("test")
    return response_base.success(data={"stored_value": value})


@router.get("/temp1", response_model=ResponseModel)
async def temp1(redis_util: RedisUtil = Depends(get_redis_util)):
    await redis_util.set("test1", "789")
    value = await redis_util.get("test1")
    return response_base.success(data={"stored_value": value})


# 登录路由
@router.post("/login", response_model=ResponseSchemaModel[LoginResponseData])
async def login(
    request: Request,
    response: Response,
    user_manager: BaseUserManager = Depends(get_user_manager), 
):
    """
    用户登录接口
    接收用户名和密码，返回JWT令牌

    Args:
        request: 请求对象
        response: 响应对象
        user_manager: 用户管理器

    Returns:
        ResponseSchemaModel: 包含访问令牌和刷新令牌的响应

    Examples:
        {
            "username": "testuser",
            "password": "testpassword"
        }
    """
    try:
        # 解析请求体中的用户名和密码
        data = await request.json()
        username = data.get("username")
        password = data.get("password")

        if not username or not password:
            return response_base.fail(
                res=CustomResponseCode.HTTP_400,
                msg="用户名和密码不能为空",
                err_code=CustomErrorCode.USER_LOGIN_FAILED,
            )

        # 验证用户凭据
        # 根据 BaseUserManager 的接口定义，authenticate 方法直接接收用户名和密码作为参数
        user = await user_manager.authenticate(username=username, password=password)

        if not user:
            return response_base.fail(
                res=CustomResponseCode.HTTP_401,
                msg="用户名或密码错误",
                err_code=CustomErrorCode.USER_LOGIN_FAILED,
            )

        if not user.status:
            return response_base.fail(
                res=CustomResponseCode.HTTP_403,
                msg="用户已禁用",
                err_code=CustomErrorCode.USER_NOT_ACTIVE,
            )

        # 创建JWT令牌
        strategy = get_redis_strategy()
        access_token = await strategy.write_token(user)

        # 更新用户的最后登录时间
        # 注意：这里需要在user_manager的on_after_login方法中实现

        return response_base.success(
            data=LoginResponseData(access_token=access_token, token_type="bearer"),
            msg="登录成功",
        )
    except Exception as e:
        print("登录失败", e)
        # return response_base.fail(res=CustomResponseCode.HTTP_500, err_code=CustomErrorCode.USER_LOGIN_FAILED)


# 注册路由（可选，根据需求决定是否启用）
@router.post("/register", response_model=ResponseSchemaModel)
async def register(
    request: Request,
    user_manager: BaseUserManager = Depends(get_user_manager),
):
    """
    用户注册接口
    接收用户名、密码等信息，创建新用户

    Args:
        request: 请求对象
        user_manager: 用户管理器

    Returns:
        ResponseSchemaModel: 注册结果

    example:
        {
            "username": "testuser",
            "password": "testpassword",
            "email": "test@example.com",
            "phone": "12345678901",
            "nickname": "测试用户"
        }
    """
    try:
        # 解析请求体
        data = await request.json()
        username = data.get("username")
        password = data.get("password")
        email = data.get("email")
        phone = data.get("phone")
        nickname = data.get("nickname", username)

        if not username or not password:
            return response_base.fail(
                res=CustomResponseCode.HTTP_400, msg="用户名和密码不能为空"
            )

        # 检查用户是否已存在
        try:
            existing_user = await user_manager.get_by_email(email) if email else None
            if existing_user:
                return response_base.fail(
                    res=CustomResponseCode.HTTP_400,
                    msg="该邮箱已被注册",
                    err_code=CustomErrorCode.USER_EXIST,
                )

            if phone:
                # 检查手机号是否已被注册（需要在user_manager中实现此方法）
                pass
        except Exception as e1:
            # 用户不存在时会抛出异常，继续执行
            print("用户不存在时会抛出异常，继续执行", e1)
            pass

        # 创建用户
        user = await user_manager.create(
            {
                "username": username,
                "password": password,
                "email": email,
                "phone": phone,
                "nickname": nickname,
                "status": True,
                "is_superuser": False,
            }
        )

        return response_base.success(data={"user_id": user.id})
    except Exception as e:
        return response_base.fail(
            res=CustomResponseCode.HTTP_500, msg=f"注册失败: {str(e)}"
        )


# 获取当前登录用户信息
@router.get("/users/me", response_model=ResponseSchemaModel)
async def get_current_user(user: SysUser = Depends(current_active_user)):
    """
    获取当前登录用户信息
    需要有效的JWT令牌

    Args:
        user: 当前登录的用户对象

    Returns:
        ResponseSchemaModel: 用户信息
    """
    return response_base.success(
        data={
            "id": user.id,
            "username": user.username,
            "nickname": user.nickname,
            "email": user.email,
            "phone": user.phone,
            "avatar": user.avatar,
            "is_superuser": user.is_superuser,
            "status": user.status,
            "last_login_at": (
                user.last_login_at.isoformat() if user.last_login_at else None
            ),
            "last_login_ip": user.last_login_ip,
        },
    )
```

 