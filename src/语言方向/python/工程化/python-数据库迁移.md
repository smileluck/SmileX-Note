- [数据库迁移](#数据库迁移)
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
  - [生产环境迁移](#生产环境迁移)
    - [1. 准备工作](#1-准备工作)
    - [2. 迁移脚本部署](#2-迁移脚本部署)
    - [3. 生产环境配置](#3-生产环境配置)
    - [4. 执行迁移的步骤](#4-执行迁移的步骤)
    - [5. 回滚方案](#5-回滚方案)
    - [6. 自动化部署集成](#6-自动化部署集成)
    - [注意事项](#注意事项)
  - [其他事项](#其他事项)
    - [关于字符编码乱码](#关于字符编码乱码)
    - [清空历史信息](#清空历史信息)


# 数据库迁移
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


## 生产环境迁移

将Alembic迁移应用到生产环境需要谨慎处理，以确保数据安全和系统稳定性。以下是一个完整的流程指南：

### 1. 准备工作
- 在开发环境中完成所有迁移脚本的测试
- 确保生产环境数据库备份可用
- 确认生产环境已安装Alembic和相关依赖

### 2. 迁移脚本部署
将开发环境中生成的迁移脚本同步到生产环境，通常通过版本控制系统（如Git）进行。

### 3. 生产环境配置
创建生产环境专用的Alembic配置，避免使用开发环境配置：

1. 需要注意调整配置文件的数据库地址
2. 这是针对 postgresql 的配置文件

```ini
# A generic, single database configuration.

[alembic]
# path to migration scripts.
# this is typically a path given in POSIX (e.g. forward slashes)
# format, relative to the token %(here)s which refers to the location of this
# ini file
script_location = %(here)s/alembic

# template used to generate migration file names; The default value is %%(rev)s_%%(slug)s
# Uncomment the line below if you want the files to be prepended with date and time
# see https://alembic.sqlalchemy.org/en/latest/tutorial.html#editing-the-ini-file
# for all available tokens
# file_template = %%(year)d_%%(month).2d_%%(day).2d_%%(hour).2d%%(minute).2d-%%(rev)s_%%(slug)s

# sys.path path, will be prepended to sys.path if present.
# defaults to the current working directory.  for multiple paths, the path separator
# is defined by "path_separator" below.
prepend_sys_path = .


# timezone to use when rendering the date within the migration file
# as well as the filename.
# If specified, requires the python>=3.9 or backports.zoneinfo library and tzdata library.
# Any required deps can installed by adding `alembic[tz]` to the pip requirements
# string value is passed to ZoneInfo()
# leave blank for localtime
# timezone =

# max length of characters to apply to the "slug" field
# truncate_slug_length = 40

# set to 'true' to run the environment during
# the 'revision' command, regardless of autogenerate
# revision_environment = false

# set to 'true' to allow .pyc and .pyo files without
# a source .py file to be detected as revisions in the
# versions/ directory
# sourceless = false

# version location specification; This defaults
# to <script_location>/versions.  When using multiple version
# directories, initial revisions must be specified with --version-path.
# The path separator used here should be the separator specified by "path_separator"
# below.
# version_locations = %(here)s/bar:%(here)s/bat:%(here)s/alembic/versions

# path_separator; This indicates what character is used to split lists of file
# paths, including version_locations and prepend_sys_path within configparser
# files such as alembic.ini.
# The default rendered in new alembic.ini files is "os", which uses os.pathsep
# to provide os-dependent path splitting.
#
# Note that in order to support legacy alembic.ini files, this default does NOT
# take place if path_separator is not present in alembic.ini.  If this
# option is omitted entirely, fallback logic is as follows:
#
# 1. Parsing of the version_locations option falls back to using the legacy
#    "version_path_separator" key, which if absent then falls back to the legacy
#    behavior of splitting on spaces and/or commas.
# 2. Parsing of the prepend_sys_path option falls back to the legacy
#    behavior of splitting on spaces, commas, or colons.
#
# Valid values for path_separator are:
#
# path_separator = :
# path_separator = ;
# path_separator = space
# path_separator = newline
#
# Use os.pathsep. Default configuration used for new projects.
path_separator = os

# set to 'true' to search source files recursively
# in each "version_locations" directory
# new in Alembic version 1.10
# recursive_version_locations = false

# the output encoding used when revision files
# are written from script.py.mako
output_encoding = utf-8

# database URL.  This is consumed by the user-maintained env.py script only.
# other means of configuring database URLs may be customized within the env.py
# file.
sqlalchemy.url = postgresql://postgres:123456@127.0.0.1:5432/cloud


[post_write_hooks]

sqlfluff.fix.encoding = utf-8
# post_write_hooks defines scripts or Python functions that are run
# on newly generated revision scripts.  See the documentation for further
# detail and examples

# format using "black" - use the console_scripts runner, against the "black" entrypoint
# hooks = black
# black.type = console_scripts
# black.entrypoint = black
# black.options = -l 79 REVISION_SCRIPT_FILENAME

# lint with attempts to fix using "ruff" - use the module runner, against the "ruff" module
# hooks = ruff
# ruff.type = module
# ruff.module = ruff
# ruff.options = check --fix REVISION_SCRIPT_FILENAME

# Alternatively, use the exec runner to execute a binary found on your PATH
# hooks = ruff
# ruff.type = exec
# ruff.executable = ruff
# ruff.options = check --fix REVISION_SCRIPT_FILENAME

# Logging configuration.  This is also consumed by the user-maintained
# env.py script only.
[loggers]
keys = root,sqlalchemy,alembic

[handlers]
keys = console

[formatters]
keys = generic

[logger_root]
level = WARNING
handlers = console
qualname =

[logger_sqlalchemy]
level = WARNING
handlers =
qualname = sqlalchemy.engine

[logger_alembic]
level = INFO
handlers =
qualname = alembic

[handler_console]
class = StreamHandler
args = (sys.stderr,)
level = NOTSET
formatter = generic

[formatter_generic]
format = %(levelname)-5.5s [%(name)s] %(message)s
datefmt = %H:%M:%S

```


### 4. 执行迁移的步骤
1. **查看状态**：确认当前数据库状态和待执行迁移
   ```bash
   alembic -c alembic_prod.ini current
   alembic -c alembic_prod.ini history --verbose
   ```

2. **测试迁移**：在执行实际迁移前进行测试
   ```bash
   alembic -c alembic_prod.ini upgrade head --sql > migration_script.sql
   ```
   检查生成的SQL脚本，确保没有问题。如果中文乱码，参考字符配置的设置默认编码。

3. **执行迁移**：正式应用迁移
   ```bash
   alembic -c alembic_prod.ini upgrade head
   ```

4. **验证迁移**：确认迁移成功
   ```bash
   alembic -c alembic_prod.ini current
   ```
   同时检查应用程序是否正常运行。

### 5. 回滚方案
准备回滚命令，以便在出现问题时快速恢复：
```bash
# 回滚到上一个版本
alembic -c alembic_prod.ini downgrade -1

# 回滚到指定版本
alembic -c alembic_prod.ini downgrade <revision_id>
```

### 6. 自动化部署集成
如果使用CI/CD流程，可以将Alembic迁移集成到部署管道中：

```python
# deploy_migrations.py
import subprocess
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def run_migrations():
    try:
        # 检查当前状态
        subprocess.run(
            ["alembic", "-c", "alembic_prod.ini", "current"],
            check=True,
            capture_output=True,
            text=True
        )
        
        # 执行迁移
        result = subprocess.run(
            ["alembic", "-c", "alembic_prod.ini", "upgrade", "head"],
            capture_output=True,
            text=True
        )
        
        if result.returncode != 0:
            logger.error(f"迁移失败: {result.stderr}")
            # 可以在这里添加自动回滚逻辑
            return False
            
        logger.info("迁移成功完成")
        return True
        
    except subprocess.CalledProcessError as e:
        logger.error(f"执行迁移时出错: {e.stderr}")
        return False

if __name__ == "__main__":
    success = run_migrations()
    exit(0 if success else 1)
```

### 注意事项
- 迁移应在低峰期执行，避免影响用户
- 对于大型数据库，考虑分阶段迁移
- 复杂迁移前一定要备份数据
- 确保应用程序代码与数据库结构版本匹配
- 多环境（开发、测试、生产）应使用不同的迁移配置

通过以上步骤，可以安全地将Alembic迁移应用到生产环境，最大程度减少风险。

## 其他事项
### 关于字符编码乱码


Alembic生成的SQL文件出现中文乱码，通常是由于编码设置不正确导致的。可以通过以下几种方式解决：

1. **在Alembic配置中指定编码**

修改`alembic_prod.ini`文件，添加编码设置：

```ini
[alembic]
# 其他配置...
output_encoding = utf-8

[post_write_hooks]
# 如果使用了文件钩子，也需要设置编码
# 例如针对sqlfluff的配置
sqlfluff.fix.encoding = utf-8
```

2. **生成SQL时指定编码**

在生成SQL文件时，通过命令行指定编码：

```bash
# Windows系统
alembic -c alembic_prod.ini upgrade head --sql | out-file -encoding utf8 migration_script.sql

# Linux/Mac系统
alembic -c alembic_prod.ini upgrade head --sql > migration_script.sql && iconv -f ASCII -t UTF-8 migration_script.sql -o migration_script.sql
```

3. **修改Alembic环境配置**

编辑`alembic/env.py`文件，确保输出编码正确：

```python
# 在env.py中找到run_migrations_online函数
def run_migrations_online():
    # 其他代码...
    
    context.configure(
        # 其他配置...
        output_encoding='utf-8',  # 添加此行
    )
```

5. **设置默认编码**

在程序入口`alembic/env.py`处添加：

```bash
import sys
import io
sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')
```
### 清空历史信息
如果你可以按照以下步骤使用 Alembic 清空记录并重新生成迁移：

1. 首先删除现有迁移记录（如果需要完全重新开始）：
```bash
# 删除 migrations 目录下 versions 文件夹中的所有迁移文件
rm -rf alembic/versions/*.py
```

2. 然后创建一个新的初始迁移：
```bash
alembic revision --autogenerate -m "Initial migration"
```

3. 如果需要清空数据库并重新应用迁移，可以使用以下步骤：
```bash
# 先降级到基础状态（如果有多个版本）
alembic downgrade base

# 然后升级到最新版本
alembic upgrade head
```

4. 如果需要彻底清空数据库表数据（保留表结构），可以在迁移脚本中添加 truncate 操作：
在生成的迁移脚本的 `upgrade()` 方法中添加：
```python
def upgrade():
    # 清空所有表数据
    op.execute("TRUNCATE TABLE your_table_name RESTART IDENTITY CASCADE;")
    # 其他自动生成的迁移代码...
```

注意：清空数据操作要非常谨慎，确保在生产环境中不会误操作。如果是开发环境需要频繁重置，可以考虑使用 fixtures 或种子数据来快速填充初始数据。