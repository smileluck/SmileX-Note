[TOC]

---

# 前言

因为项目中使用到了image-webpack-loader 进行图片压缩，然后运行yarn run serve，出现问题。
![在这里插入图片描述](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a78a4136bcb4ddca1e3e7642843a348~tplv-k3u1fbpfcp-zoom-1.image)
根据错误分析出，缺少模块gifsicle。

# 解决思路

1. 进入node_modules
   1. 查找文件夹/image-webpack-loader是否存在，存在查看optionalDependencies依赖信息，发现使用imagemin-gifsicle。
   2. 查看文件夹/imagemin-gifsicle是否存在，发现正常存在/bin,/lib文件夹等。
   3. 删除node_modules重新安装，错误依然存在
2. 使用yarn add单独安装
   
   ```bash
   yarn add imagemin-jpegtran imagemin-svgo imagemin-gifsicle imagemin-optipng --dev
   ```
   
   此时出现问题

```bash
getaddrinfo ENOENT raw.githubusercontent.com
```

3. ping raw.githubusercontent.com 发现无法访问
4. 使用第三方ip工具https://site.ip138.com/raw.Githubusercontent.com/后，获得可访问ip
5. 打开hosts文件，将代码填入尾部。
   
   ```bash
   151.101.76.133 raw.githubusercontent.com
   ```
6. 再次执行yarn add单独安装，安装成功
7. 执行yarn run serve，正常。问题解决

---

老规矩写的不清晰的地方，可以指出，我会不定时更新和修改。一起加油.

所有的伟大，都来自于一个勇敢的开始。
