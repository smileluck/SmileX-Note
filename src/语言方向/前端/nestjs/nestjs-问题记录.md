
# ReferenceError: crypto is not defined when using TypeORM in Nest.js 
- 问题说明：当使用新版本nestjs/TYPEOREM时，会出现 `ReferenceError: crypto is not defined when using TypeORM in Nest.js `，这是因为nodejs版本不匹配
- 解决办法：使用 `node>20` 以上的版本既可。


# Cannot find module '.prisma/client/default'
- 版本： prisma 6.10+
- 异常信息   Error: Cannot find module '.prisma/client/default'

1. 配置 schema.prisma
```ini
generator client {
  provider = "prisma-client-js"
  output   = "./generated/client"
}
```
2. 运行 prisma generate
3. 配置路径别名
```ts
{
    "compilerOptions": {
        "paths": {
            "@prisma-client": ["./prisma/generated/client"],
        }
    }
}
```
4. 配置nestjs打包
```json
{
    "compilerOptions": {
        "deleteOutDir": true,
        "watchAssets": true,
        "assets": [
            {
                "include": "../prisma/generated",
                "outDir": "dist/prisma",
                "watchAssets": true
            }
        ],
    }
}
```
5. 修改引入
```ts
import { PrismaClient } from  "@prisma/client";
// 更改为
import { PrismaClient } from  "@prisma-client";
```