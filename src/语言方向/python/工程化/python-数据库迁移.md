
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
    - [方式二：自动扫描导入指定文件的模型文件](#方式二自动扫描导入指定文件的模型文件)
    - [方式三：使用Alembic插件（进阶：需额外安装依赖）](#方式三使用alembic插件进阶需额外安装依赖)
    - [推荐插件：`alembic-autoimport-models`](#推荐插件alembic-autoimport-models)
    - [优点](#优点)


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


#### 方式二：自动扫描导入指定文件的模型文件
```python

import os
import importlib


# 自动扫描并导入所有模型模块
def auto_import_models():
    PROJECT_ROOT = os.path.abspath(os.path.join(os.path.dirname(__file__), ".."))
    # 模型所在的根目录（根据实际项目结构调整）

    # 需要扫描的目录
    include_dirs = ["app/models"]

    # 排除不需要扫描的目录（可选）
    exclude_dirs = ["__pycache__", "tests"]

    for include_dir in include_dirs:
        models_root_dir = os.path.join(PROJECT_ROOT, include_dir)
        if os.path.exists(models_root_dir):
            # 递归扫描所有Python文件
            for root, dirs, files in os.walk(models_root_dir):
                # 过滤排除目录
                dirs[:] = [d for d in dirs if d not in exclude_dirs]

                for file in files:
                    # 只处理.py文件，且排除__init__.py
                    if file.endswith(".py") and not file.startswith("__"):
                        # 构造模块的相对路径
                        relative_path = os.path.relpath(root, PROJECT_ROOT)
                        module_name = (
                            relative_path.replace(os.sep, ".") + "." + file[:-3]
                        )

                        try:
                            # 动态导入模块（导入后模型会自动注册到Base.metadata）
                            importlib.import_module(module_name)
                            # 可选：打印导入的模块名，方便调试
                            print(f"已自动导入模型模块: {module_name}")
                        except ImportError as e:
                            # 非致命错误：打印警告但不中断程序
                            print(f"警告: 无法导入模型模块 {module_name}，错误: {e}")

# 执行自动导入
auto_import_models()
```


#### 方式三：使用Alembic插件（进阶：需额外安装依赖）
如果上述两种方式仍不满足需求，可使用社区维护的Alembic插件，通过配置实现模型自动导入。但这种方式依赖第三方库，需权衡兼容性。

#### 推荐插件：`alembic-autoimport-models`
该插件可通过配置文件指定模型目录，自动扫描并导入继承`Base`的模型，无需修改`env.py`。

1. **安装插件**：
   ```bash
   pip install alembic-autoimport-models
   ```

2. **配置`alembic.ini`**：
   在`alembic.ini`中添加插件和模型目录配置：
   ```ini
   # alembic.ini
   [alembic]
   script_location = alembic
   # 启用插件
   plugins = alembic_autoimport_models
   # 指定模型所在的包（多个目录用逗号分隔）
   autoimport_models = core.models, other.package.models
   # 指定Base类的路径（插件需找到Base来识别模型）
   autoimport_base = core.base:Base
   ```

3. **简化`env.py`**：
   插件会自动导入模型，`env.py`只需保留`Base`的导入：
   ```python
   # alembic/env.py
   from logging.config import fileConfig
   from sqlalchemy import engine_from_config, pool
   from alembic import context
   from core.base import Base  # 只需导入Base，插件自动加载模型

   config = context.config
   fileConfig(config.config_file_name)
   target_metadata = Base.metadata  # 插件已确保metadata包含所有模型

   # ... 迁移函数（不变）
   ```

#### 优点
- **零代码修改**：无需手动写导入或聚合文件，通过配置实现自动化。
- **支持多目录**：可同时指定多个模型目录，插件自动扫描。

