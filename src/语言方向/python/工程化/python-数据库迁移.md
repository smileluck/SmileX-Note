
- [alembic](#alembic)
  - [1. 安装 Alembic](#1-安装-alembic)
  - [2. 初始化 Alembic 环境](#2-初始化-alembic-环境)
  - [3. 配置数据库连接](#3-配置数据库连接)
  - [4. 创建第一个迁移脚本](#4-创建第一个迁移脚本)
  - [5. 编辑迁移脚本](#5-编辑迁移脚本)
  - [6. 应用迁移](#6-应用迁移)
  - [7. 回滚迁移](#7-回滚迁移)
  - [8. 自动生成迁移脚本](#8-自动生成迁移脚本)
  - [常用命令总结](#常用命令总结)
  - [无法识别到类时，可以主动扫描注入](#无法识别到类时可以主动扫描注入)
    - [方式一：自动扫描导入所有模型文件](#方式一自动扫描导入所有模型文件)


## alembic

Alembic 是 SQLAlchemy 作者开发的数据库迁移工具，用于管理数据库模式的版本控制和迁移。它可以帮助你在开发过程中安全地修改数据库结构，同时保持不同环境（开发、测试、生产）之间的数据库一致性。

以下是使用 Alembic 的基本步骤和示例：

### 1. 安装 Alembic
```bash
pip install alembic
```

### 2. 初始化 Alembic 环境
在项目根目录执行：
```bash
alembic init alembic
```

这会创建一个 `alembic` 目录和 `alembic.ini` 配置文件。

### 3. 配置数据库连接
编辑 `alembic.ini` 文件，设置数据库连接字符串：
```ini
sqlalchemy.url = postgresql://user:password@localhost/dbname
# 或者对于 SQLite
# sqlalchemy.url = sqlite:///mydatabase.db
```

### 4. 创建第一个迁移脚本
```bash
alembic revision -m "initial migration"
```

这会在 `alembic/versions` 目录下创建一个新的迁移脚本，文件名格式为 `xxxxxx_initial_migration.py`。

### 5. 编辑迁移脚本
打开生成的迁移脚本，在 `upgrade()` 和 `downgrade()` 方法中定义数据库结构变更：

```python
"""initial migration

Revision ID: xxxxxx
Revises: 
Create Date: 2025-09-02 10:00:00.000000

"""
from alembic import op
import sqlalchemy as sa


# revision identifiers, used by Alembic.
revision = 'xxxxxx'
down_revision = None
branch_labels = None
depends_on = None


def upgrade() -> None:
    # 创建用户表
    op.create_table(
        'users',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('name', sa.String(50), nullable=False),
        sa.Column('email', sa.String(100), unique=True, nullable=False),
        sa.Column('created_at', sa.DateTime, default=sa.func.now())
    )


def downgrade() -> None:
    # 回滚操作：删除用户表
    op.drop_table('users')

```



### 6. 应用迁移
将迁移应用到数据库：
```bash
alembic upgrade head
```

### 7. 回滚迁移
如果需要回滚到上一个版本：
```bash
alembic downgrade -1
```

### 8. 自动生成迁移脚本
Alembic 可以根据模型变化自动生成迁移脚本：

```python
# 在 env.py 中找到 target_metadata 并修改
from myapp.models import Base  # 导入你的模型基类
target_metadata = Base.metadata
```



然后运行：
```bash
alembic revision --autogenerate -m "add new fields to users table"
```

这会自动检测模型与数据库的差异并生成迁移脚本。

### 常用命令总结
- `alembic init <directory>`: 初始化迁移环境
- `alembic revision -m "message"`: 创建新的迁移脚本
- `alembic revision --autogenerate -m "message"`: 自动生成迁移脚本
- `alembic upgrade head`: 应用所有未应用的迁移
- `alembic upgrade +1`: 应用下一个迁移
- `alembic downgrade -1`: 回滚上一个迁移
- `alembic current`: 显示当前迁移版本
- `alembic history`: 显示迁移历史

使用 Alembic 可以有效地管理数据库结构的变更，特别适合团队协作和多环境部署的项目。


### 无法识别到类时，可以主动扫描注入
在 `alembic/env.py` 中添加以下代码，查看识别的表
```python
print("已识别的表：", target_metadata.tables.keys())
```


#### 方式一：自动扫描导入所有模型文件
```python

import os
import importlib

# 模型所在的根目录（根据实际项目结构调整）
MODEL_DIR = os.path.join(os.path.dirname(__file__), "../app/models")

# 自动扫描并导入所有模型文件
for root, dirs, files in os.walk(MODEL_DIR):
    for file in files:
        if file.endswith(".py") and not file.startswith("__"):
            # 构造模块路径（例如：core.models.user）
            relative_path = os.path.relpath(root, os.path.dirname(MODEL_DIR))
            module_name = (
                f"app.{relative_path.replace(os.sep, '.')}.{file[:-3]}"
            )
            print(relative_path)
            try:
                # 导入模块（会自动注册模型到Base.metadata）
                importlib.import_module(module_name)
            except ImportError as e:
                print(f"警告：无法导入模型模块 {module_name}，错误：{e}")
```
