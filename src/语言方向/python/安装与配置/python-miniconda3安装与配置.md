[toc]

---

# MiniConda3安装与配置

> 下载地址： https://docs.conda.io/projects/miniconda/en/latest/

1. 下载并安装 `miniconda`

2. 测试是否安装成功

   ```shell
   conda --version
   
   conda config --show # 查看所有配置信息
   ```

3. 配置显示通道URL。（win：C:/users/【用户名】/.condarc；linux：~/.condarc）

   ```python3
   conda config --set show_channel_urls yes
   ```

   win 系统下，此时才会出现 `.condarc` 文件

4. 配置是否进入 `base` 环境

    ```shell
    # 关闭自动进入base环境
    conda config --set auto_activate_base false
    
    # 开启自动进入base环境
    conda config --set auto_activate_base true
    ```

    

5. 打开 `.condarc` 文件

   - 清华源

     ```config
      channels:
        - defaults
      show_channel_urls: true
      default_channels:
        - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
        - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
        - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
      custom_channels:
        conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
        msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
        bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
        menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
        pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
        pytorch-lts: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
        simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
        deepmodeling: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
     ```

     或另一种方式，提高优先级

     ```config
     channels:
       - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
       - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
       - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
       - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
       - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
       - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/menpo/
       - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
       - defaults
     show_channel_urls: true
     
     ```

   - 交大源

     ```config
     channels:
       - defaults
     show_channel_urls: true
     default_channels:
       - https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/main/
       - https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/free/
       - https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud/msys2/
     custom_channels:
       conda-forge: https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud
       msys2: https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud
       bioconda: https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud
       menpo: https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud
       pytorch: https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud
       simpleitk: https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud
     ```

     或另一种方式，高优先级：

     ```config
     channels:
       - https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/main/
       - https://mirrors.sjtug.sjtu.edu.cn/anaconda/pkgs/free/
       - https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud/conda-forge/
       - https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud/msys2/
       - https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud/bioconda/
       - https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud/menpo/
       - https://mirrors.sjtug.sjtu.edu.cn/anaconda/cloud/pytorch/
       - defaults
     show_channel_urls: true
     ```

6. ```shell
    conda config --get channels                    # 获取已有的通道
    conda config –set show_channel_urls yes       # 搜索时显示通道地址
    ```

7. 修改包保存路径。

   ```config
   envs_dirs:
     - E://conda//pkg  # 包存储路径
   ```