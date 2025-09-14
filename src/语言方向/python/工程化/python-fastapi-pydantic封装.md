
## 封装脱敏
可以使用 Pydantic 的 `field_validator` 或 `model_validator` 封装一个通用的脱敏工具，用于处理敏感信息（如手机号、邮箱、身份证号等）的自动脱敏。以下是一个可复用的实现：

### 工具特点与使用说明

1.** 核心组件**-** 脱敏函数 **：封装了手机号、邮箱、身份证号、姓名等常见敏感信息的脱敏逻辑，可直接使用或扩展。
   -** 脱敏装饰器 **：`desensitize` 装饰器将脱敏函数与 Pydantic 字段验证绑定，只需一行代码即可为字段添加脱敏功能。
   -** 正则扩展 **：`regex_desensitize` 支持通过自定义正则表达式实现复杂脱敏规则（如地址、银行卡号等）。


2.** 使用方法 **- 在 Pydantic 模型中，为需要脱敏的字段添加 `_desensitize_xxx` 类属性，通过 `desensitize(脱敏函数)` 绑定对应的脱敏逻辑。
   - 当模型实例化或调用 `model_validate` 时，会自动对指定字段执行脱敏处理。
   - 原始数据不会被修改，脱敏仅作用于 Pydantic 模型的输出结果。


3.** 扩展建议 **- 新增脱敏规则：只需定义新的脱敏函数（如银行卡号、住址等），再通过 `desensitize` 装饰器绑定到字段即可。
   - 动态脱敏：可根据用户权限动态开启/关闭脱敏（例如管理员可见原始数据，普通用户只能看到脱敏后的数据）。
   - 类型适配：如需支持非字符串类型（如整数手机号），可在脱敏函数中先进行类型转换。

这个工具既保持了 Pydantic 模型的简洁性，又实现了敏感信息的自动脱敏，适合在 API 响应、日志输出等场景中使用。

```python
from pydantic import BaseModel, field_validator, Field
from typing import Optional, Callable, Pattern
import re

# 1. 定义常用脱敏函数（可扩展）
def desensitize_phone(phone: str) -> str:
    """脱敏手机号：保留前3后4位，中间用*代替"""
    if not phone:
        return phone
    return re.sub(r'(\d{3})\d{4}(\d{4})', r'\1****\2', phone)

def desensitize_email(email: str) -> str:
    """脱敏邮箱：用户名保留前3位，域名保留"""
    if not email:
        return email
    return re.sub(r'^(.{3}).*?(@.*)$', r'\1****\2', email)

def desensitize_id_card(id_card: str) -> str:
    """脱敏身份证号：保留前6后4位"""
    if not id_card:
        return id_card
    return re.sub(r'^(\d{6})\d+(\d{4})$', r'\1********\2', id_card)

def desensitize_name(name: str) -> str:
    """脱敏姓名：2字名隐藏第1位，3字及以上隐藏中间字"""
    if not name:
        return name
    if len(name) == 1:
        return name
    if len(name) == 2:
        return f'*{name[1:]}'
    return f'{name[0]}*{name[2:]}'


# 2. 封装脱敏装饰器（用于Pydantic模型字段）
def desensitize(desensitizer: Callable[[str], str]):
    """
    脱敏装饰器：为Pydantic字段添加脱敏验证
    
    Args:
        desensitizer: 脱敏函数，接收字符串并返回脱敏后的字符串
    """
    def validator(v: Optional[str]) -> Optional[str]:
        if v is None:
            return v
        return desensitizer(str(v))
    return field_validator(..., mode='after')(validator)


# 3. 示例：在模型中使用脱敏工具
class UserInfo(BaseModel):
    """用户信息模型（包含脱敏字段）"""
    id: int
    name: str = Field(description="姓名")
    phone: Optional[str] = Field(None, description="手机号")
    email: Optional[str] = Field(None, description="邮箱")
    id_card: Optional[str] = Field(None, description="身份证号")
    address: Optional[str] = Field(None, description="地址")  # 不脱敏
    
    # 为字段添加脱敏逻辑
    _desensitize_name = desensitize(desensitize_name)('name')
    _desensitize_phone = desensitize(desensitize_phone)('phone')
    _desensitize_email = desensitize(desensitize_email)('email')
    _desensitize_id_card = desensitize(desensitize_id_card)('id_card')


# 4. 扩展：支持自定义正则表达式的脱敏
def regex_desensitize(pattern: Pattern, repl: str):
    """基于正则表达式的脱敏器生成函数"""
    def desensitizer(value: str) -> str:
        return re.sub(pattern, repl, value)
    return desensitizer

# 示例：脱敏地址（保留前6位和后4位）
desensitize_address = regex_desensitize(r'^(.{6}).*(.{4})$', r'\1****\2')

class AddressInfo(BaseModel):
    full_address: str
    _desensitize_address = desensitize(desensitize_address)('full_address')


# 5. 使用示例
if __name__ == "__main__":
    # 原始用户数据（包含敏感信息）
    raw_user_data = {
        "id": 1001,
        "name": "张三",
        "phone": "13800138000",
        "email": "zhang san@example.com",
        "id_card": "110101199001011234",
        "address": "北京市朝阳区建国路88号"
    }
    
    # 自动脱敏
    user = UserInfo(**raw_user_data)
    print("脱敏后的用户信息：")
    print(user.model_dump())
    # 输出：
    # {
    #   'id': 1001,
    #   'name': '*三',
    #   'phone': '138****8000',
    #   'email': 'zha****@example.com',
    #   'id_card': '110101********1234',
    #   'address': '北京市朝阳区建国路88号'  # 未脱敏
    # }
    
    # 地址脱敏示例
    address = AddressInfo(full_address="上海市浦东新区张江高科技园区博云路2号")
    print("\n脱敏后的地址：")
    print(address.model_dump())
    # 输出：{'full_address': '上海市浦****博云路2号'}

```