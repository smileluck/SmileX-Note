[toc]

---

# electron+electron forge+vite+react+ts

>  How to create an Electron app with React, TypeScript, and Electron Forge 

## 环境说明

1. 安装最新的node环境。这里使用 18.16.0 

## 开始构建

1. 使用 `electron forge` 搭建 `vite` 集成的框架

```shell
npm init electron-app@latest my-new-app -- --template=vite
```

2. 集成 `react` 依赖

```shell
npm install --save react react-dom
npm install --save-dev @types/react @types/react-dom
npm install --save-dev @vitejs/plugin-react
```

3. 集成`tailwind css`

   1. 安装 `tailwind css`

	```shell
	npm install -D tailwindcss postcss autoprefixer
	npx tailwindcss init -p
	```
	
	2. 配置 `tailwind.config.js` 。如果没有在根目录下创建
	
	```js
	/** @type {import('tailwindcss').Config} */
	export default {
	  content: [
	    "./index.html",
	    "./src/**/*.{js,ts,jsx,tsx}",
	  ],
	  theme: {
	    extend: {},
	  },
	  plugins: [],
	}
	```
	
	3. 在 `index.css` 添加如下代码，或者全局引用的`css`文件
	
	```css
	@tailwind base;
	@tailwind components;
	@tailwind utilities;
	```

4. 集成 `react-router v6`

```shell
npm install react-router-dom localforage match-sorter sort-by
```

