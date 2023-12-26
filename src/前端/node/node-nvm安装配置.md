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

