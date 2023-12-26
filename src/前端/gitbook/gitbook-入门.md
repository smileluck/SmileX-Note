[toc]

---

# 安装

1. 安装 `node 12+`

2. 配置淘宝镜像

   ```shell
   npm config set registry https://registry.npm.taobao.org
   ```

3. 安装全局 `gitbook-cli`

   ```shell
   npm install gitbook-cli -g
   ```

## 问题记录

### TypeError: cb.apply is not a function

找到对应的文件 `D:\dev-tools\nvm-noinstall\v14.18.2\node_modules\gitbook-cli\node_modules\npm\node_modules\graceful-fs\polyfills.js` 注释下面几行代码

```shell

  // fs.stat = statFix(fs.stat)
  // fs.fstat = statFix(fs.fstat)
  // fs.lstat = statFix(fs.lstat)
```



# 常用指令

1. 初始化 `gitbook`。

   - 初始化项目，按照 `gitbook` 规范会自动创建 `README.md` 和 `SUMMARY.md` 两个文件，具体用途见下文.
   - 其实 `SUMMARY.md` 是电子书的章节目录，`gitbook` 会初始化相应的文件目录结构，所以主要是用于**开发初始阶段**.

   ```shell
   gitbook init
   ```

2. 启动 `gitbook`

   - 启动本地服务，程序无报错则可以在浏览器预览电子书效果: [http://localhost:4000](https://www.oschina.net/action/GoToLink?url=http%3A%2F%2Flocalhost%3A4000)
   - 由于能够实时预览电子书效果，并且大多数开发环境搭建在本地而不是远程服务器中，所以主要用于**开发调试阶段**.

   ```shell
   gitbook serve
   ```

3. 构建 `gitbook` 静态网页

   - 构建静态网页而不启动本地服务器，默认生成文件存放在 `_book/` 目录，当然输出目录是可配置的，暂不涉及，见高级部分.
   - 输出静态网页后可打包上传到服务器，也可以上传到 `github` 等网站进行托管，因而主要用于**发布准备阶段**.

   ```shell
   gitbook build
   ```



# 部署

## nginx映射

```conf
location /book {
    alias /usr/local/serve/frontend/gitbook/;
    index index.html;
}
```

