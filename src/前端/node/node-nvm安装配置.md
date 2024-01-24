[toc]

---

# win安装及配置

1. 安装包下载

[安装包]: https://github.com/coreybutler/nvm-windows

下载 `noinstall` 版本

2. 配置环境变量
   1. 系统->高级系统设置->环境变量->系统变量
   2. 增加以下配置
      1. `NVM_HOME`：nvm-window安装路径。`D:\dev-tools\nvm-noinstall`
      2. `NVM_SYMLINK`：nvm-window的node存储路径。`D:\repository\node`
      3. `PATH`配置下增加 `%NVM_SYMLINK%` 和 `%NVM_HOME%`
3. 进入nvm-window的安装路径，在根目录添加文件 `settings.txt`，文件内容如下：

```
root: D:\dev-tools\nvm-noinstall # 根目录
path: D:\repository\node # node存储路径
arch: 64 # 操作系统
proxy: none # 代理
originalpath: .
originalversion: 
node_mirror: http://npm.taobao.org/mirrors/node/ # node镜像源
npm_mirror: https://npm.taobao.org/mirrors/npm/ # npm镜像源
```

4. 配置全局安装配置并配置**环境变量**。

    ```shell
    npm config set prefix "D:\dev\repositories\node_global"
    ```

    - 环境变量配置 `NODE_PATH`：`D:\dev\repositories\node_global\node_modules`
    - PATH里面添加：`D:\dev\repositories\node_global`

1. 配置缓存位置

```shell
npm config set cache "D:\dev\repositories\node_cache"
```

# 安装pnpm

1. 进入全局安装配置文件夹 `D:\dev\repositories\node_global`

2. 执行命令

   ```shell
   # linux || MacOS
   curl -fsSL https://get.pnpm.io/install.sh | sh -
   
   # window 在PowerShell操作
   Invoke-WebRequest 'https://get.pnpm.io/v6.16.js' -UseBasicParsing -o pnpm.js; node pnpm.js add --global pnpm; Remove-Item pnpm.js
   ```

# 常用指令

- `nvm install 【版本】`。安装指定版本。
- `nvm uninstall 【版本】`。卸载指定版本
- `nvm list `。显示已经安装的版本
- `nvm list available`。显示可用版本。
- `nvm use [版本]`。切换指定版本
- `nvm current`。查看当前版本