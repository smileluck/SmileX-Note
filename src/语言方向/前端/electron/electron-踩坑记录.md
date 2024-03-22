[toc]

---

# electron: Running postinstall script, failed in 44.4s

1. 配置镜像源

   ```shell
   # 配置
   pnpm config set electron_mirror https://npm.taobao.org/mirrors/electron/
   pnpm config set electron_mirror http://npm.taobao.org/mirrors/electron/
   
   
   # 删除 
   pnpm config delete electron_mirror
   ```

2. 执行安装 

   ```shell
   pnpm install 
   ```

   