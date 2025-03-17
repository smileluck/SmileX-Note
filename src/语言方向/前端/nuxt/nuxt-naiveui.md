[toc]

---

# Nuxt.js + Naive UI

## 方案一

> nuxtjs-naive-ui

1. 安装依赖
```shell
# npm
npx nuxi module add nuxtjs-naive-ui

# pnpm
pnpm dlx nuxi module add nuxtjs-naive-ui

# unplugin-vue-components
pnpm install unplugin-vue-components -D
```

2. 配置 nuxt.config.ts

Autoimport 可以酌情使用

```ts
import AutoImport from 'unplugin-auto-import/vite'
import { NaiveUiResolver } from 'unplugin-vue-components/resolvers'
import Components from 'unplugin-vue-components/vite'

// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  modules: ['nuxtjs-naive-ui'],
  vite: {
    plugins: [
      AutoImport({
        imports: [
          {
            'naive-ui': [
              'useDialog',
              'useMessage',
              'useNotification',
              'useLoadingBar'
            ]
          }
        ]
      }),
      Components({
        resolvers: [NaiveUiResolver()]
      })
    ]
  }
})
```


## 方案二

> naive-ui 
> @css-render/vue3-ssr
> unplugin-vue-components

1. 安装依赖
```shell

pnpm install naive-ui

pnpm install naive-ui @css-render/vue3-ssr unplugin-vue-components -D

```

2. 配置naive-ui.ts
解决 css 资源首次未加载，和 nuxtjs-naive-ui 会有冲突，导致异常提示 ` App already provides property with key "@css-render/vue3-ssr". It will be overwritten with the new value. `
```ts
import { setup } from '@css-render/vue3-ssr'

export default defineNuxtPlugin((nuxtApp) => {
  const { collect } = setup(nuxtApp.vueApp)
  useServerHead({
    style: () => {
      const stylesString = collect()
      const stylesArray = stylesString.split(/<\/style>/g).filter(style => style)
      return stylesArray.map((styleString: string) => {
        const match = styleString.match(/<style cssr-id="([^"]*)">([\s\S]*)/)
        if (match) {
          const id = match[1]
          return { 'cssr-id': id, children: match[2] }
        }
        return {}
      })
    }
  })
})
```

3. 配置 nuxt.config.ts

```ts
import { NaiveUiResolver } from 'unplugin-vue-components/resolvers'
import Components from 'unplugin-vue-components/vite'

// https://nuxt.com/docs/api/configuration/nuxt-config
export default defineNuxtConfig({
  // components: true, // 开启组件自动导入
  alias: {
    "@components": "/components",
    "@assets": "/assets",
    "@utils": "/utils"
  },
  compatibilityDate: '2024-04-03',
  devServer: {
    host: '127.0.0.1'
  },
  devtools: { enabled: false },
  modules: [
    '@unocss/nuxt',
    '@nuxtjs/i18n',
    '@nuxt/content',
    'nuxt-toc',
    '@pinia/nuxt',
    '@nuxt/icon'
  ],
  css: [
    '/assets/css/reset.css'
  ],
  vue: {
    compilerOptions: {
      isCustomElement: (tag) => tag === "iconify-icon",
    },
  },
  content: {
    highlight: {
      theme: 'github-dark'
    },
    markdown: {
      toc: {
        depth: 3,
        searchDepth: 3
      }
    }
  },
  unocss: {
    nuxtLayers: true,
  },
  plugins: [
    '~/plugins/naive-ui.ts',
    '~/plugins/storage.ts',
  ],
  vite: {
    optimizeDeps: {
      include:
        process.env.NODE_ENV === 'development'
          ? ['naive-ui', '@css-render/vue3-ssr']
          : []
    },
    plugins: [
      Components({
        dirs: ['components'], // 配置需要自动导入的组件目录
        resolvers: [NaiveUiResolver()]
      })
    ]
  },
  build: {
    transpile: process.env.NODE_ENV === 'production'
      ? [
        'naive-ui',
        '@css-render/vue3-ssr',
      ]
      : []
  },
  i18n: {
    strategy: 'no_prefix',
    locales: [
      { code: 'en-US', language: 'en-US', name: 'English' },
      { code: 'zh-CN', language: 'zh-CN', name: '中文' },
    ],
    defaultLocale: 'en-US',
    vueI18n: './i18n.config.ts',
    detectBrowserLanguage: {
      // alwaysRedirect: true,
      useCookie: true,
      cookieKey: 'lang',
      cookieCrossOrigin: true,
      redirectOn: 'root' // recommended
    }
  }
})
```