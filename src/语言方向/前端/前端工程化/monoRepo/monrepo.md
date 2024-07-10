[toc]

---

# 简介

monorepo  多包管理框架主要是将多个项目的代码放到一个文件夹中，便于同时管理、开发。一个monorepo仓库可以包含多个项目，也可以包含多个项目的不同模块，同一个项目拆分为不同模块不仅可以方便团队进行管理，也方便不同团队、不同项目之间多次复用，在多个系统中实现样式及标准的统一。目前来说比较流行。

# 搭建monorepo

1. 首先需要有一个包管理工具，这里使用 `pnpm`，

   1. pnpm相较于npm、yarn可以有效节省磁盘空间并提升安装速度，性能对比[查看](https://link.zhihu.com/?target=https%3A//www.pnpm.cn/benchmarks)。
   2. pnpm内置了对单个代码仓库包含多个软件包的支持，是monorepo架构模式的不二速选。

2. 在新建一个文件夹 `packages`，并进行初始化 `pnpm init`。初始化后会在当前文件夹下添加 `package.json` 

3. 在根目录下的添加对应的 `sub-package`，有两种方式

   1. 基于`package.json`，增加 `workspaces` 字段

      ```json
      {
        "name": "uno-ui",
        "private": true,
        "version": "0.0.0",
        "type": "module",
        "scripts": {
          "dev": "vite",
          "build": "vue-tsc && vite build",
          "preview": "vite preview"
        },
        "dependencies": {
          "vue": "^3.4.21"
        },
        "devDependencies": {
          "@vitejs/plugin-vue": "^5.0.4",
          "typescript": "^5.2.2",
          "vite": "^5.2.0",
          "vue-tsc": "^2.0.6"
        },
        "workspaces": [
          "packages/*"
        ],
      }
      
      ```

   2. 基于 `pnpm-workspace.yaml`

      ```yaml
      packages:
        - "packages/*"
      ```

# 注意点

1. 根目录的`package.json`已有的依赖可以不用在字项目重复过添加。所以可以共同依赖放到根目录中。
2. 如果项目存在相互引用，需要注意打包时的配置，设计文件 `package.json`、`vite.config.ts`

# 文章记录

【vite+pnpm+vue3】https://zhuanlan.zhihu.com/p/592894426

【monorepo】https://blog.csdn.net/qq_42345136/article/details/137670505

【pnpm】 https://www.pnpm.cn/cli/add