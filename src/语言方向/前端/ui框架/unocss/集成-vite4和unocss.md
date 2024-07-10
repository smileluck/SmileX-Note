[toc]

---

# 前言

## 什么是原子化

https://antfu.me/posts/reimagine-atomic-css-zh

# UNOCSS

 [**UnoCSS**](https://github.com/antfu/unocss) - 具有高性能且极具灵活性的即时原子化 CSS 引擎。 

## 使用和配置

UnoCSS 是一个引擎，而非一款框架，因为它并未提供核心工具类，所有功能可以通过预设和内联配置提供。

- 规则(Rules) - 定义原子 CSS 实用程序
- 预设(Presets) - 常见用例的预定义配置。
- 快捷方式(Shortcuts) - 将多个规则组合成一个简写。
- 主题(Theme) - 定义主题变量。
- 变体(Variants) - 将自定义约定应用于规则。
- 转换器(Transformers) - 将转换器编码为用户源代码以支持约定。
- 提取器(Extractors) - 定义提取实用程序的位置和方式。
- 预检(Preflights) - 定义全局 CSS 规则。
- 层(Layers) - 定义每个实用程序层的顺序。
- 自动完成(AutoComplete) - 定义自定义自动完成建议。

## unocss集成

### vite+unocss

- unocss. The Vite plugin ships with the `unocss` package. 

```shell
pnpm add -D unocss
```

- install plugin

```js
import vue from '@vitejs/plugin-vue'
import UnoCSS from 'unocss/vite'
import { defineConfig } from 'vite'

import path from 'path'
// https://vitejs.dev/config/
export default defineConfig(({ command, mode }) => {
  return {
    // 项目插件
    plugins: [
      vue(),
      UnoCSS()
    ],
    // 基础配置
    base: './',
    publicDir: 'public',
    resolve: {
      alias: {
        '@': path.resolve(__dirname, 'src'),
      },
    },
    css: {
      preprocessorOptions: {
        less: {
          modifyVars: {
            '@border-color-base': '#dce3e8',
          },
          javascriptEnabled: true,
        },
      },
    },
    build: {
      outDir: 'dist',
      assetsDir: 'assets',
      assetsInlineLimit: 4096,
      cssCodeSplit: true,
      brotliSize: false,
      sourcemap: false,
      minify: 'terser',
      terserOptions: {
        compress: {
          // 生产环境去除console及debug
          drop_console: true,
          drop_debugger: true,
        },
      },
    },
  }
})

```

- Create a  `uno.config.js` file:

```js
// uno.config.ts
import { defineConfig } from 'unocss'

export default defineConfig({
    content: {
        pipeline: {
            exclude: ['node_modules', 'dist']
        }
    },
    theme: {
        fontSize: {
            'icon-xs': '0.875rem',
            'icon-small': '1rem',
            icon: '1.125rem',
            'icon-large': '1.5rem',
            'icon-xl': '2rem'
        }
    },
    presets: [
        presetAttributify({ /* preset options */ }),
        presetUno(),
    ],
})
```

- Add `virtual:uno.css` to your main entry

```js
// main.ts
import 'virtual:uno.css'
```



### style reset

UnoCSS does not provide style resetting or preflight by default so not to populate your global CSS and also for maximum flexibility. If you use UnoCSS along with other CSS frameworks, they probably already do the resetting for you. If you use UnoCSS alone, you can use resetting libraries like [Normalize.css](https://github.com/csstools/normalize.css).

- install

```shell
pnpm add @unocss/reset
```

## preset

### iconify

> icon引用网站
>
> https://icones.js.org/
>
> https://icon-sets.iconify.design/

- install

```shell
pnpm add -D @unocss/preset-icons 

pnpm add -D @iconify-json/[the-collection-you-want]

# If you prefer to install all the icon sets available on Iconify at once (~130MB):

pnpm add -D @iconify/json
```

- config

```js
// uno.config.ts
import { defineConfig } from 'unocss'
import presetIcons from '@unocss/preset-icons'

export default defineConfig({
  presets: [
    presetIcons({ /* options */ }),
    // ...other presets
  ],
})
```

- 按需加载(Bundler)

当使用捆绑器时，你可以使用动态导入提供一个集合，如此他们将被打包为异步块并按需加载。

```js
//uno.config.js	
import presetIcons from '@unocss/preset-icons/browser'

export default defineConfig({
  presets: [
    presetIcons({
      collections: {
        carbon: () => import('@iconify-json/carbon/icons.json').then(i => i.default),
        mdi: () => import('@iconify-json/mdi/icons.json').then(i => i.default),
        logos: () => import('@iconify-json/logos/icons.json').then(i => i.default),
      }
    })
  ]
})
```

### Attributify

>  This enables the [attributify mode](https://unocss.dev/presets/attributify#attributify-mode) for other presets. 

```shell
pnpm add -D @unocss/preset-attributify
```

```js
// uno.config.ts
import presetAttributify from '@unocss/preset-attributify'

export default defineConfig({
  presets: [
    presetAttributify({ /* preset options */ }),
    // ...
  ],
})
```

