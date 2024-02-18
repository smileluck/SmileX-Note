[toc]

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
  pip freeze 			# 显示pip安装的包及版本库
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

  