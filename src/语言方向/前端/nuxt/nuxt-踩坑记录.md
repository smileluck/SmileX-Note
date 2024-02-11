[toc]

---

# nuxt2

## jsconfig.json is not in cwd 

> npm generate 报错，提示 jsconfig.json is not in cwd。

解决方法：

1. 打开node_modules/@nuxt/cli/dist/cli-generate.js
2. 用 ` cwd: upath . normalize ( rootDir ) ` 替换 ` cwd: rootDir ` 。

```javascript

  const files = await globby__default['default']('**/*.*', {
    ...globbyOptions,
    ignore,
    cwd: rootDir, // 这行替换掉
    absolute: true
  });


  const files = await globby__default['default']('**/*.*', {
    ...globbyOptions,
    ignore,
    cwd: upath . normalize ( rootDir ),
    absolute: true
  });

```