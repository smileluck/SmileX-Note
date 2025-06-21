
# 前言
使用vite+vue3进行ssr渲染，采用预渲染模式实现SSG

前端预渲染（Frontend prerendering）是一种技术，它可以在浏览器渲染 HTML页面之前，将页面预先渲染成静态HTML文件，并将其保存在服务器上。当用户访问网站时，服务器直接提供预先渲染的静态HTML文件，而不是动态生成 HTML 页面。
这种技术的优点在于可以大大提高网站的加载速度和搜索引擎优化效果。由于预渲染的 HTML 文件是静态的，可以被搜索引擎轻易地爬取和索引，从而提高网站的搜索排名和可见性。

# 使用 vite-prerender-plugin

> GitHub: https://github.com/preactjs/vite-prerender-plugin/tree/master/examples

1. 安装
```shell
pnpm i vite-prerender-plugin
```
2. 配置 vite.config.ts
```javascript
import { defineConfig } from 'vite'
import prerender from 'vite-prerender-plugin'

export default defineConfig({
  plugins: [
    vitePrerenderPlugin({
        renderTarget: "#app",
        prerenderScript: "./prerender.js"
    })
  ],
})
```
3. 配置prerender.js
```javascript
export async function prerender() {
    return {
        html:"<h1>hello</h1>",
        links: new Set(["/", "/about"]),
        head: {
            title: "xxxx",
            elements: new Set([
                { type: 'meta', props: { name: "keywords", property: 'keywords', content: 'xxxx' } },
                { type: 'meta', props: { name: "description", property: 'description', content: 'xxxx' } },
            ]),
        }
    };
}
```

# 另外的方法（未尝试）
 
## prerenderer插件
里面主要有两个包大家可以根据自己项目需求进行选择。
@prerenderer/renderer-jsdom  ： 使用jsdom，速度快，但不可靠，不能处理高级用法。可能无法与所有的前端框架和应用程序一起工作。
@prerenderer/renderer-puppeteer : 使用puppeteer在无头的Chrome中渲染页面。比之前的ChromeRenderer更简单、更可靠。

```shell
npm i -D @prerenderer/rollup-plugin @prerenderer/renderer-puppeteer
```

```javascript
import { defineConfig } from 'vite'
import prerender from '@prerenderer/rollup-plugin'
const prerenderConfig = {
  routes: [
    "/",
    "/about/member",
  ],
  renderer: "@prerenderer/renderer-puppeteer",
  rendererOptions: {
    renderAfterDocumentEvent: "custom-render-trigger",
    // headless: false
  },
  postProcess(renderedRoute) {
    console.log(renderedRoute.route);
    path.join(__dirname, "dist", renderedRoute.route);

    renderedRoute.route = renderedRoute.originalRoute;
    renderedRoute.html = renderedRoute.html
      .replace(/http:/i, "https:")
      .replace(
        /(https:\/\/)?(localhost|127\.0\.0\.1):\d*/i,
        process.env.CI_ENVIRONMENT_URL || ""
      );
  },
};
export default defineConfig({
  plugins: [
    prerender(prerenderConfig),
  ],
});
```



作者：想赚点零花钱
链接：https://juejin.cn/post/7225435922946719799
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
