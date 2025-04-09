[TOC]

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

# nuxt3

## Nuxt3中使用SSR模式下有UI显示BUG

> Nuxt3中使用SSR模式下有UI显示BUG.
> 
> 使用naiveUI+nuxt3.7
> 
> 效果：刷新会显示原始样式然后再出现正确样式

解决办法：[Feature: return an object with id keys and style values for vue3-ssr · Issue #1108 · 07akioni/css-render · GitHub](https://github.com/07akioni/css-render/issues/1108)

安装依赖：

```shell
pnpm add 
```

在`plugins/naive-ui.ts`中添加如下代码：

```typescript
mport { setup } from '@css-render/vue3-ssr'

export default defineNuxtPlugin((nuxtApp: any) => {
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



## nuxt3使用ssr+naiveui生产模式构建有问题
> Named export 'VResizeObserver' not found. The requested module 'vueuc' is a CommonJS module,

解决办法：https://github.com/tusen-ai/naive-ui/issues/4641

```typescript

export default defineNuxtConfig({
	vite: {
		ssr: {
			// https://github.com/histoire-dev/histoire/issues/488
			noExternal: ["naive-ui"], // 添加即可
		},
	}
})
```