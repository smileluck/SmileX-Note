[toc]

---

# prisma-迁移数据库

## 从数据模式映射到数据库

```shell
npx prisma migrate dev --name init
```

## 从数据库同步到数据模式

1. 初始化 prisma 设置
```shell
npx prisma init
```
2. 生成 prisma 模型
```shell
npx prisma generate
```
1. 从数据库同步
```shell
npx prisma db pull
```
1. 推送到数据库
```shell
npx prisma db push
```


## migrate 数据库迁移
| 命令                  | 作用                                                                 |
|-----------------------|----------------------------------------------------------------------|
| `prisma migrate dev`  | 创建新迁移文件并应用到数据库（开发环境）。                           |
| `prisma migrate deploy` | 应用所有未执行的迁移（生产环境）。                                  |
| `prisma migrate reset` | 重置数据库（删除所有数据并重新应用所有迁移，仅用于开发）。         |
| `prisma migrate status` | 检查迁移状态（哪些迁移已应用，哪些未应用）。                        |
| `prisma db push`      | 直接将 schema 变更同步到数据库（不生成迁移文件，适合快速迭代）。    |


- 与 prisma db push 的对比
    - prisma migrate。适合生产环境，提供完整的迁移历史记录，确保数据库变更可重现、可审计。
    - prisma db push。适合开发阶段，快速同步 schema 变更，但不记录迁移历史，不建议用于生产环境。