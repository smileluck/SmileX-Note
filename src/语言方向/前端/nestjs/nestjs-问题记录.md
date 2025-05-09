
# ReferenceError: crypto is not defined when using TypeORM in Nest.js 
- 问题说明：当使用新版本nestjs/TYPEOREM时，会出现 `ReferenceError: crypto is not defined when using TypeORM in Nest.js `，这是因为nodejs版本不匹配
- 解决办法：使用 `node>20` 以上的版本既可。

