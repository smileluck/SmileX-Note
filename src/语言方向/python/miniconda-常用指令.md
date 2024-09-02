[TOC]

---

# 基本指令

- 查看 `conda` 版本
  
  ```shell
  conda --version
  ```

- 更新最新版 `conda`
  
  ```shell
  conda update conda
  ```

- 查看当前环境所有包
  
  ```shell
  conda list
  ```

- 查询 包有那些版本。 
  
  ```shell
  conda search [包名]
  ```

- 更新当前环境所有包和指定包到最新版本
  
  ```shell
  conda update --all
  conda update [package_name]
  ```

- 安装包在当前环境，需要先激活环境，否则装到默认环境
  
  ```shell
  conda install [package_name]
  conda install [package_name]=1.0.0 # 指定版本包
  ```

- 安装到指定环境
  
  ```shell
  conda install -n [env] [package_name]
  ```

- 删除包在当前环境
  
  ```shell
  conda remove [package_name]
  ```

# 环境相关

- 查看虚拟环境
  
  ```shell
  conda env list
  ```

- 创建虚拟环境。 conda create -n [环境名] python=[版本号]
  
  ```shell
  conda create -n python_3.9 python=3.9
  ```

- 复制环境。 conda create --name [复制后新名] --clone [被复制环境名]
  
  ```shell
  conda create --name Py_3.9 --clone python_3.9
  ```

- 激活虚拟环境
  
  ```shell
  conda activate python_3.9
  ```

- 退出虚拟环境
  
  ```shell
  conda deactivate
  ```

- 删除虚拟环境。 conda remove -n [虚拟名] --all 
  
  ```shell
  conda remove -n python_3.9 --all
  ```
