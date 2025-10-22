- [poetry配置镜像](#poetry配置镜像)
  - [**1. 临时使用镜像源（单次安装）**](#1-临时使用镜像源单次安装)
  - [**2. 永久配置镜像源**](#2-永久配置镜像源)
    - [方法一：修改 Poetry 全局配置](#方法一修改-poetry-全局配置)
    - [方法二：直接编辑配置文件](#方法二直接编辑配置文件)
  - [**3. 常用 PyPI 镜像源**](#3-常用-pypi-镜像源)
  - [**4. 恢复默认源**](#4-恢复默认源)


## poetry配置镜像

在 Poetry 中配置镜像源可以加快依赖包的安装速度，尤其适合网络环境受限的情况。以下是配置镜像源的方法：


### **1. 临时使用镜像源（单次安装）**
安装包时通过 `--source` 指定镜像源（需先添加源）：
```bash
# 先添加镜像源（以清华PyPI镜像为例）
poetry source add --priority=default tsinghua https://pypi.tuna.tsinghua.edu.cn/simple

# 安装包时使用该源
poetry add 包名 --source tsinghua
```


### **2. 永久配置镜像源**
#### 方法一：修改 Poetry 全局配置
```bash
# 设置默认镜像源（清华镜像为例）
poetry config repositories.tsinghua https://pypi.tuna.tsinghua.edu.cn/simple

# 将镜像源设为优先
poetry config pypi.org.mirror-url https://pypi.tuna.tsinghua.edu.cn/simple
```

#### 方法二：直接编辑配置文件
Poetry 配置文件位置：
- Windows：`C:\Users\用户名\AppData\Roaming\pypoetry\config.toml`
- macOS/Linux：`~/.config/pypoetry/config.toml`

添加或修改以下内容：
```toml
[repositories]
tsinghua = { url = "https://pypi.tuna.tsinghua.edu.cn/simple" }

[pypi]
mirror-url = "https://pypi.tuna.tsinghua.edu.cn/simple"
```


### **3. 常用 PyPI 镜像源**
- 清华：`https://pypi.tuna.tsinghua.edu.cn/simple`
- 阿里云：`https://mirrors.aliyun.com/pypi/simple/`
- 豆瓣：`https://pypi.doubanio.com/simple/`
- 华为云：`https://mirrors.huaweicloud.com/repository/pypi/simple/`


### **4. 恢复默认源**
如果需要切换回官方源：
```bash
poetry config --unset pypi.org.mirror-url
```


配置后，Poetry 会优先从镜像源下载依赖，大幅提升国内环境下的安装速度。如果某个包在镜像源中找不到，Poetry 会自动尝试官方源。