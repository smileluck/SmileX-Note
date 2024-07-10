[toc]

---



# 扁平化配置

 ESLint迎来了一次重大更新，9.0.0版本，根据[官方文档](https://eslint.nodejs.cn/docs/latest/use/getting-started)介绍，使用新版的先决条件是[Node.js](https://so.csdn.net/so/search?q=Node.js&spm=1001.2101.3001.7020)版本必须是18.18.0、20.9.0，或者是>=21.1.0的版本，新版ESLint将不再直接支持以下旧版配置(非扁平化)文件： 

扁平化配置是引入的默认配置格式，最初作为实验性功能出现在 ESLint v8.21.0 中，并在 ESLint v9 中变得稳定并成为默认设置。

快速区分参考：

- 扁平化配置：`eslint.config.js`、`eslint.config.mjs`
- 旧版配置：`.eslintrc`、`.eslintrc.json`、`.eslintrc.js`

**为什么选择扁平配置？**

旧版 `eslintrc` 格式是在 JavaScript 早期设计的，那时 ES 模块和现代 JavaScript 特性尚未标准化。涉及许多隐含习惯惯例，并且 `extends` 功能导致最终配置结果难以理解和预测，这也使得共享配置难以维护和调试。

```json
{
  "extends": [
    "@nuxtjs",
    "plugin:vue/vue3-recommended",
  ],
  "rules": {
    // ...
  }
}
```

新的扁平化配置将插件和配置的解析从 ESLint 内部约定移动到原生 ES 模块解析，这使得它更加明确透明，甚至可以从其他模块导入。由于扁平化配置只是一个 JavaScript 模块，它还为更多自定义打开了大门。

