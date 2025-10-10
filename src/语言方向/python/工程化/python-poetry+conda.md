- [现有项目](#现有项目)
  - [1. 准备工作](#1-准备工作)
  - [2. 在现有项目中初始化 Poetry](#2-在现有项目中初始化-poetry)
  - [3. 配置 Poetry 复用 Conda 环境](#3-配置-poetry-复用-conda-环境)
  - [4. 导入现有依赖到 Poetry](#4-导入现有依赖到-poetry)
  - [5. 处理 Conda 特有的依赖](#5-处理-conda-特有的依赖)
  - [6. 验证集成结果](#6-验证集成结果)
  - [7. 后续工作流](#7-后续工作流)

## 现有项目
将已有的 FastAPI 项目集成 Poetry 并保留 Conda 环境中的依赖包，可以按照以下步骤操作，确保平滑过渡且不破坏现有依赖：

### 1. 准备工作
确保当前工作在正确的 Conda 环境中：
```bash
# 激活现有 Conda 环境
conda activate your-existing-env

# 确认当前环境的 Python 路径（后续会用到）
which python  # Linux/macOS
where python  # Windows
```

### 2. 在现有项目中初始化 Poetry
进入你的 FastAPI 项目目录，初始化 Poetry：
```bash
# 进入项目目录
cd your-fastapi-project

# 初始化 Poetry（使用 --no-interaction 可跳过交互式提问）
poetry init --no-interaction
```

这会生成 `pyproject.toml` 文件，后续将通过它管理依赖。

### 3. 配置 Poetry 复用 Conda 环境
关键步骤：让 Poetry 使用当前激活的 Conda 环境，而非创建新虚拟环境：
```bash
# 配置 Poetry 不自动创建虚拟环境
poetry config virtualenvs.create false

# 验证配置（确保 virtualenvs.create = false）
poetry config --list
```

### 4. 导入现有依赖到 Poetry
将 Conda 环境中已安装的依赖（尤其是 FastAPI 相关）添加到 Poetry：


    


### 5. 处理 Conda 特有的依赖
如果项目中使用了只能通过 Conda 安装的包（如 `numpy`、`pytorch` 等）：
1. 先通过 Conda 确保它们已安装：`conda install -c conda-forge 包名`
2. 在 `pyproject.toml` 中标记为外部依赖（避免 Poetry 尝试安装）：
   ```toml
   [tool.poetry]
   # ... 其他配置

   [tool.poetry.dependencies]
   python = "^3.10"
   fastapi = "^0.103.1"
   uvicorn = "^0.23.2"
   # 对于Conda安装的包，指定版本但标记为可选（Poetry不会强制安装）
   numpy = {version = "1.25.2", optional = true}
   ```

### 6. 验证集成结果
```bash
# 检查依赖是否正确安装
poetry show

# 运行 FastAPI 项目验证
poetry run uvicorn main:app --reload
```

如果一切正常，项目会像以前一样运行，但依赖管理已切换到 Poetry。

### 7. 后续工作流
- 新增依赖：`poetry add 包名`（Python 包）或 `conda install 包名`（系统级包）
- 提交代码时，确保包含 `pyproject.toml` 和 `poetry.lock`
- 团队成员协作：先 `conda env create -f environment.yml` 恢复 Conda 环境，再 `poetry install` 安装 Python 依赖

这种方式既保留了 Conda 处理复杂依赖的优势，又获得了 Poetry 对 Python 包的精细化版本管理能力，非常适合 FastAPI 项目的依赖管理升级。