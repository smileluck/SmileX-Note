[toc]

---

# 前言

`pnpm` 支持 `monorepo` 模式的工作机制叫做 [workspace(工作空间)](https://link.zhihu.com/?target=https%3A//pnpm.io/zh/workspaces)。

它要求在代码仓的根目录下存有 `pnpm-workspace.yaml` 文件指定哪些目录作为独立的工作空间，这个工作空间可以理解为一个子模块或者 `npm` 包。

例如以下的 `pnpm-workspace.yaml` 文件定义：`a` 目录、`b` 目录、`c` 目录下的所有子目录，都会各自被视为独立的模块。

```text
packages:
  - a
  - b
  - c/*
my-project
 ┣  a
 ┃ ┗  package.json
 ┣  b
 ┃ ┗  package.json
 ┣  c
 ┃ ┣  c-1
 ┃ ┃ ┗  package.json
 ┃ ┣  c-2
 ┃ ┃ ┗  package.json
 ┃ ┗  c-3
 ┃   ┗  package.json
 ┣  package.json
 ┣  pnpm-workspace.yaml
```

`pnpm` 并**不是通过目录名称，而是通过目录下** `**package.json**` **文件的** `**name**` **字段来识别仓库内的包与模块的。**

# 中枢管理操作

在 `workspace` 模式下，项目根目录通常不会作为一个子模块或者 `npm` 包，而是_主要作为一个管理中枢_，执行一些全局操作，安装一些共有的依赖，每个子模块都能访问根目录的依赖，适合把 `TypeScript`、`Vite`、`eslint` 等公共开发依赖装在这里，下面简单介绍一些常用的中枢管理操作。

- 创建一个 `package.json` 文件。

```csharp
pnpm init
```

在项目跟目录下运行 `pnpm install`，pnpm 会根据当前目录 `package.json` 中的依赖声明安装全部依赖，**在 workspace 模式下会一并处理所有子模块的依赖安装**。

- 安装项目公共开发依赖，声明在根目录的 package.json - devDependencies 中。`-w` 选项代表在 `monorepo` 模式下的根目录进行操作。

```arduino
// 安装
pnpm install -wD xxx
// 卸载
pnpm uninstall -w xxx
```

- 执行根目录的 package.json 中的脚本

```arduino
pnpm run xxx
```

# 子包管理操作

在 `workspace` 模式下，`pnpm` 主要通过 `--filter` 选项过滤子模块，实现对各个工作空间进行精细化操作的目的。

1. **为指定模块安装外部依赖。**
2. 下面的例子指为 `a` 包安装 `lodash` 外部依赖。
3. 同样的道理，`-S` 和 `-D` 选项分别可以将依赖安装为正式依赖(`dependencies`)或者开发依赖(`devDependencies`)。

```css
// 为 a 包安装 lodash 
pnpm --filter a i -S lodash // 生产依赖
pnpm --filter a i -D lodash // 开发依赖
```

1. **指定内部模块之间的互相依赖。**
2. 指定模块之间的互相依赖。下面的例子演示了为 `a` 包安装内部依赖 `b`。

```css
// 指定 a 模块依赖于 b 模块
pnpm --filter a i -S b
```

`pnpm workspace` 对内部依赖关系的表示不同于外部，它自己约定了一套 [Workspace 协议 (workspace:)](https://link.zhihu.com/?target=https%3A//pnpm.io/zh/workspaces%23workspace-%E5%8D%8F%E8%AE%AE-workspace)。下面给出一个内部模块 `a` 依赖同是内部模块 `b` 的例子。

```json
{
  "name": "a",
  // ...
  "dependencies": {
    "b": "workspace:^"
  }
}
```

在实际发布 `npm` 包时，`workspace:^` 会被替换成内部模块 `b` 的对应版本号(对应 `package.json` 中的 `version` 字段)。替换规律如下所示：

```json
{
  "dependencies": {
    "a": "workspace:*", // 固定版本依赖，被转换成 x.x.x
    "b": "workspace:~", // minor 版本依赖，将被转换成 ~x.x.x
    "c": "workspace:^"  // major 版本依赖，将被转换成 ^x.x.x
  }
}
```