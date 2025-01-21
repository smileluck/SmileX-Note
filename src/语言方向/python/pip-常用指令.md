[TOC]

---

# 常用指令

- 查看版本
  
  ```shell
  pip --version
  ```

- 升级 pip
  
  ```shell
  pip install -U pip
  ```

- 搜索包版本
  
  ```shell
  pip search [package_name]
  ```

- 查看已安装的包
  
  ```shell
  pip list
  pip list --outdatea # 列出所有过期的库
  pip freeze             # 显示pip安装的包及版本库
  pip freeze > d:/out.txt # 写入到文件
  ```

- 安装包
  
  ```shell
  pip install [package_name]                      # 最新版本
  pip install [package_name]==1.21.2              # 指定版本
  pip install [package_name]>=1.21.2              # 最小版本
  pip install [package_name] --ignore-installed   # 忽略 numpy 包是否已安装，都将重新安装
  ```

- 从 `requirements.txt` 安装
  
  ```shell
  pip install -r requirements.txt
  ```

- 升级包
  
  ```shell
  pip install --upgrade [package_name]
  ```

- 下载包
  
  ```shell
  pip uninstall [package_name]
  ```

- 显示包信息
  
  ```shell
  pip show [package_name]                 # 显示包的详情
  pip show -f [package_name]              # 显示包所在目录
  ```

# 生成requirements

要生成一个只包含项目实际依赖的 `requirements.txt` 文件，可以使用 `pipreqs` 工具。`pipreqs` 会根据项目中的 `import` 语句自动检测所需的依赖包，而不是列出当前环境中所有已安装的包。以下是具体步骤：

## 使用 `pipreqs` 生成 `requirements.txt`

### 1. 安装 `pipreqs`

首先，你需要安装 `pipreqs`。可以使用 `pip` 来安装：

```bash
pip install pipreqs
```

### 2. 生成 `requirements.txt`

在你的项目根目录下运行以下命令：

```bash
pipreqs .
```

这将在当前目录下生成一个 `requirements.txt` 文件，包含项目中实际使用的依赖包。

### 3. （可选）指定文件路径

如果你想将 `requirements.txt` 生成到特定目录，可以指定路径：

```bash
pipreqs /path/to/your/project
```

## 示例

假设你的项目结构如下：

```
my_project/
├── main.py
├── utils.py
└── ...
```

在 `my_project` 目录下运行：

```bash
pipreqs .
```

生成的 `requirements.txt` 可能如下所示：

```
numpy==1.21.2
pandas==1.3.3
requests==2.26.0
```

## 注意事项

- **虚拟环境**：建议在虚拟环境中运行 `pipreqs`，以确保只检测项目相关的依赖，而不包括全局或其他项目的依赖。
  
  ```bash
  python -m venv venv
  source venv/bin/activate  # Linux/MacOS
  venv\Scripts\activate     # Windows
  ```

- **自定义依赖**：有时 `pipreqs` 可能无法检测到某些通过配置文件（如 `setup.py` 或 `pyproject.toml`）安装的依赖。在这种情况下，你可能需要手动添加这些依赖到 `requirements.txt`。

- **版本控制**：确保将 `requirements.txt` 添加到版本控制系统（如 Git）中，以便团队成员能够轻松安装相同的依赖。

## 其他方法

如果你不想使用 `pipreqs`，还可以手动检查项目中的 `import` 语句，并创建 `requirements.txt` 文件。这种方法虽然繁琐，但在某些情况下可能更准确。

---

通过以上方法，你可以生成一个只包含项目实际依赖的 `requirements.txt` 文件，确保项目的依赖关系清晰且易于管理。


