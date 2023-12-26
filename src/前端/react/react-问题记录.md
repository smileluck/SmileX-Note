[toc]

---

# Cannot find module './index.module.css' or its corresponding type declarations 

> 基于vite+react+electron 使用 css module

要导入自定义文件类型，必须使用 `TypeScript` 的`declare module`语法让它知道可以导入。为此，只需在其他代码的根目录所在的位置创建一个`globals.d.ts`（声明文件），然后添加以下代码：

```tsx
declare module '*.css';
```

配置配置文件 `tsconfig.json`

```json
{
    "compilerOptions": {
        "target": "ESNext",
        "module": "ESNext",
        "lib": [
            "dom",
            "dom.iterable",
            "ESNext"
        ],
        "skipLibCheck": true,
        /* Bundler mode */
        "allowJs": true,
        "esModuleInterop": true,
        "allowSyntheticDefaultImports": true,
        "forceConsistentCasingInFileNames": true,
        "moduleResolution": "node", //node环境
        "resolveJsonModule": true,
        "isolatedModules": true,
        "noEmit": true,
        "jsx": "react-jsx",
        "baseUrl": "src",
        // "paths": {
        //     "@/*": [
        //         "./*"
        //     ]
        // },
        "experimentalDecorators": true,
        /* Linting */
        "strict": true,
        "noUnusedLocals": true,
        "noUnusedParameters": true,
        "noFallthroughCasesInSwitch": true
    },
    "include": [
        "src",
        "globals.d.ts" //配置的.d.ts文件
    ]
}
```



