- [python-uv 基础](#python-uv-基础)
  - [安装和使用](#安装和使用)
    - [一、前提准备：安装 uv](#一前提准备安装-uv)
    - [二、创建项目并初始化环境](#二创建项目并初始化环境)
      - [1. 创建项目（可选，快速生成项目骨架）](#1-创建项目可选快速生成项目骨架)
      - [2. 初始化虚拟环境](#2-初始化虚拟环境)
      - [3. 激活虚拟环境（不同系统命令）](#3-激活虚拟环境不同系统命令)
      - [4. 初始化依赖（可选，添加基础依赖）](#4-初始化依赖可选添加基础依赖)
    - [三、核心配置文件说明](#三核心配置文件说明)
    - [总结](#总结)
- [uv 配置系统特定标识与创建不同依赖组笔记](#uv-配置系统特定标识与创建不同依赖组笔记)
  - [一、配置系统特定标识（环境标记）](#一配置系统特定标识环境标记)
    - [核心作用](#核心作用)
    - [1. 基础语法格式](#1-基础语法格式)
    - [2. 常用配置场景](#2-常用配置场景)
      - [（1）按平台限制依赖安装](#1按平台限制依赖安装)
      - [（2）按平台配置不同依赖源](#2按平台配置不同依赖源)
      - [（3）多环境多源配置](#3多环境多源配置)
      - [（4）按环境配置不同索引（如 PyTorch CPU/GPU 区分）](#4按环境配置不同索引如-pytorch-cpugpu-区分)
    - [3. 标记组合使用](#3-标记组合使用)
  - [二、创建不同依赖组](#二创建不同依赖组)
    - [核心概念](#核心概念)
    - [1. 核心依赖组类型及配置](#1-核心依赖组类型及配置)
      - [（1）开发依赖组（默认组：`dev`）](#1开发依赖组默认组dev)
      - [（2）自定义依赖组（如 `lint`、`test`）](#2自定义依赖组如-linttest)
    - [2. 依赖组使用命令](#2-依赖组使用命令)
    - [3. 高级配置](#3-高级配置)
      - [（1）修改默认组](#1修改默认组)
      - [（2）组依赖兼容性](#2组依赖兼容性)
      - [（3）旧版兼容（`dev-dependencies`）](#3旧版兼容dev-dependencies)
    - [4. 依赖组与可选依赖的区别](#4-依赖组与可选依赖的区别)
  - [三、关键注意事项](#三关键注意事项)


# python-uv 基础

## 安装和使用

你想要了解的是如何使用 Python 的 `uv` 工具来创建项目并初始化开发环境，这是一个比传统 `pip`/`venv` 更高效的现代包管理工具。

### 一、前提准备：安装 uv
首先确保你的系统中已经安装了 `uv`，如果没有，执行以下命令：
```bash
# macOS/Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"

# 验证安装
uv --version
```

### 二、创建项目并初始化环境
`uv` 提供了简洁的命令来完成项目创建和环境初始化，核心流程分为「创建项目」「初始化虚拟环境」「管理依赖」三步：

#### 1. 创建项目（可选，快速生成项目骨架）
`uv` 支持快速生成标准化的项目结构（包含 `pyproject.toml` 等核心文件）：
```bash
# 创建名为 my_project 的项目（会自动生成基础目录和配置）
uv init my_project

# 进入项目目录
cd my_project
```
执行后会生成如下结构：
```
my_project/
├── my_project/       # 项目源码目录
│   └── __init__.py
├── pyproject.toml    # 项目配置文件（核心，管理依赖/构建）
├── README.md
└── tests/            # 测试目录
    └── __init__.py
```

#### 2. 初始化虚拟环境
`uv` 会自动管理虚拟环境，也可以手动指定创建/激活：

```bash
# 方式1：自动创建并使用虚拟环境（推荐，无需手动激活）
# uv 会在项目根目录生成 .venv 文件夹，后续命令自动使用该环境
uv venv

# 方式2：指定大版本（例如 Python 3.11，uv 会选最新的 3.11.x）
uv venv --python 3.11

# 方式3：指定精确版本（例如 3.11.8）
uv venv --python 3.11.8

# 方式4：简写参数 -p（和 --python 等效，更简洁）
uv venv -p 3.10

# 方式5：指定已有虚拟环境
# macOS/Linux 示例（指定系统已安装的 Python 3.9 路径）
uv venv -p /usr/bin/python3.9

# Windows 示例（指定本地 Python 3.8 路径）
uv venv -p C:\Python38\python.exe

```

#### 3. 激活虚拟环境（不同系统命令）

```bash
# macOS/Linux
source .venv/bin/activate
# Windows (Cmd)
.venv\Scripts\activate.bat
# Windows (PowerShell)
.venv\Scripts\Activate.ps1
```

#### 4. 初始化依赖（可选，添加基础依赖）
创建环境后，可通过 `uv` 安装依赖并写入配置文件：
```bash
# 安装依赖（例如安装 requests），并自动更新 pyproject.toml
uv add requests

# 安装开发依赖（例如 pytest），仅用于开发环境
uv add --dev pytest

# 生成锁定文件（锁定依赖版本，保证环境一致性）
uv lock

# 从 pyproject.toml 安装所有依赖（多人协作/部署时使用）
uv sync
```

### 三、核心配置文件说明
生成的 `pyproject.toml` 是核心配置文件，初始内容如下（关键部分已注释）：
```toml
[project]
name = "my_project"          # 项目名
version = "0.1.0"            # 版本
dependencies = [             # 生产环境依赖
  "python >= 3.8",           # Python 版本要求
  # "requests >= 2.31.0"     # 安装依赖后自动添加
]

[tool.uv]
# uv 专属配置，指定虚拟环境路径
virtualenv = ".venv"

[project.optional-dependencies]
dev = [                      # 开发环境依赖
  # "pytest >= 7.0"          # 安装开发依赖后自动添加
]
```

### 总结
1. **核心流程**：安装 uv → `uv init 项目名` 创建项目 → `uv venv` 初始化虚拟环境 → `uv add` 管理依赖。
2. **关键优势**：`uv` 无需手动激活环境（大部分命令自动使用 `.venv`），依赖安装/解析速度远快于传统工具。
3. **核心文件**：`pyproject.toml` 统一管理项目配置和依赖，`uv lock` 生成的 `uv.lock` 保证环境一致性。


# uv 配置系统特定标识与创建不同依赖组笔记
## 一、配置系统特定标识（环境标记）
### 核心作用
通过环境标记（遵循 Python 环境标记规范），限制依赖项仅在特定平台、Python 版本等环境下安装，或为不同环境配置不同依赖源。

### 1. 基础语法格式
依赖项后添加 `; 环境标记条件`，支持的核心标记（完整列表参考 Python 官方文档）：
- `sys_platform`：操作系统（如 `linux`、`darwin`（macOS）、`win32`（Windows））
- `python_version`：Python 版本（如 `"3.11"`、`"3.9"`）
- `platform_system`：系统名称（如 `"Darwin"`、`"Linux"`、`"Windows"`）

### 2. 常用配置场景
#### （1）按平台限制依赖安装
示例1：仅在 Linux 安装 `jax`
```bash
uv add 'jax; sys_platform == "linux"'
```
生成的 `pyproject.toml` 配置：
```toml
[project]
dependencies = ["jax; sys_platform == 'linux'"]
```

示例2：仅在 Python 3.11+ 安装 `numpy`
```bash
uv add 'numpy; python_version >= "3.11"'
```

#### （2）按平台配置不同依赖源
示例：macOS 从 GitHub 安装 `httpx`，其他平台从 PyPI 安装
```toml
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = {
  git = "https://github.com/encode/httpx",
  tag = "0.27.2",
  marker = "sys_platform == 'darwin'"  # 仅 macOS 生效
}
```

#### （3）多环境多源配置
示例：macOS 和 Linux 使用不同版本的 `httpx` Git 依赖
```toml
[project]
dependencies = ["httpx"]

[tool.uv.sources]
httpx = [
  {
    git = "https://github.com/encode/httpx",
    tag = "0.27.2",
    marker = "sys_platform == 'darwin'"  # macOS
  },
  {
    git = "https://github.com/encode/httpx",
    tag = "0.24.1",
    marker = "sys_platform == 'linux'"   # Linux
  }
]
```

#### （4）按环境配置不同索引（如 PyTorch CPU/GPU 区分）
```toml
[project]
dependencies = ["torch"]

[tool.uv.sources]
torch = [
  { index = "torch-cpu", marker = "platform_system == 'Darwin'" },  # macOS 用 CPU 版
  { index = "torch-gpu", marker = "platform_system == 'Linux'" }   # Linux 用 GPU 版
]

[[tool.uv.index]]
name = "torch-cpu"
url = "https://download.pytorch.org/whl/cpu"

[[tool.uv.index]]
name = "torch-gpu"
url = "https://download.pytorch.org/whl/cu124"
```

### 3. 标记组合使用
支持 `and`、`or` 和括号组合条件：
```bash
uv add 'aiohttp >=3.7.4,<4; (sys_platform != "win32" or implementation_name != "pypy") and python_version >= "3.10"'
```

## 二、创建不同依赖组
### 核心概念
依赖组用于分类管理依赖（如开发、 lint、测试等），仅本地生效，发布时不包含（除 `project.dependencies` 外），遵循 PEP 735 标准。

### 1. 核心依赖组类型及配置
#### （1）开发依赖组（默认组：`dev`）
- 作用：存放开发阶段所需依赖（如测试工具 `pytest`）
- 配置命令：
  ```bash
  uv add --dev pytest  # 添加到 dev 组
  ```
- 生成的 `pyproject.toml` 配置：
  ```toml
  [dependency-groups]
  dev = ["pytest >=8.1.1,<9"]  # 自动添加版本约束
  ```
- 特性：默认同步（`uv run`/`uv sync` 自动包含），支持 `--only-dev`（仅同步 dev 组）、`--no-dev`（排除 dev 组）

#### （2）自定义依赖组（如 `lint`、`test`）
- 作用：按功能分类依赖（如代码检查 `ruff`、文档生成工具）
- 配置命令：
  ```bash
  uv add --group lint ruff  # 添加到 lint 组
  uv add --group test pytest-cov  # 添加到 test 组
  ```
- 生成的配置：
  ```toml
  [dependency-groups]
  dev = ["pytest"]          # 原 dev 组
  lint = ["ruff"]           # 自定义 lint 组
  test = ["pytest-cov"]     # 自定义 test 组
  ```

### 2. 依赖组使用命令
| 命令作用 | 命令示例 |
|----------|----------|
| 仅同步指定组 | `uv sync --only-group lint` |
| 排除指定组 | `uv sync --no-group test` |
| 同时同步多个组 | `uv sync --group dev --group lint` |
| 从指定组删除依赖 | `uv remove ruff --group lint` |

### 3. 高级配置
#### （1）修改默认组
默认同步 `dev` 组，可通过 `tool.uv.default-groups` 调整：
```toml
[tool.uv]
default-groups = ["dev", "lint"]  # 默认同步 dev 和 lint 组
```

#### （2）组依赖兼容性
- `uv` 要求所有依赖组相互兼容，冲突时解析失败（需显式声明冲突依赖）
- 示例：`dev` 组和 `lint` 组的依赖版本需兼容

#### （3）旧版兼容（`dev-dependencies`）
- 旧版使用 `tool.uv.dev-dependencies` 配置开发依赖，会与 `dependency-groups.dev` 合并
- 配置示例（不推荐，将被弃用）：
  ```toml
  [tool.uv]
  dev-dependencies = ["pytest"]  # 与 dev 组合并
  ```

### 4. 依赖组与可选依赖的区别
| 类型 | 用途 | 配置位置 | 发布时是否包含 |
|------|------|----------|----------------|
| 依赖组（如 dev、lint） | 本地开发分类（测试、 lint 等） | `[dependency-groups]` | 不包含 |
| 可选依赖（extras） | 库的可选功能（如 pandas[excel]） | `[project.optional-dependencies]` | 包含（需用户显式安装） |

## 三、关键注意事项
1. 环境标记中的版本号需用引号括起来（如 `python_version >= "3.11"`）
2. 依赖源的环境标记仅影响源的选择，依赖项本身会在所有平台显示（仅源按条件生效）
3. 依赖组的 `--dev` 本质是 `--group dev` 的快捷方式
4. 发布包时，建议用 `uv build --no-sources` 禁用自定义源，确保兼容性