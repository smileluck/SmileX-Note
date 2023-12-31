[toc]

---

# 前言

目前多租户SaaS架构在市面上比较流行，俨然已经成为趋势，而多租户SaaS架构的核心在于让多用户环境下 使用同 一套程序，且保证用户间的数据隔离，其中数据隔离是重点。  



# SAAS 多租户技术组件

## 基于Web
> 前端请求中默认带上x-tenant-id，值为用户编号

## Security 层
### 基于 Shiro
> 获取登陆用户的租户ID，检验是否有权限访问该租户

### 基于 Spring Security

> 获取登陆用户的租户ID，检验是否有权限访问该租户

### 分布式下认证中心

## DataBase层
基于数据库的实现方式，可以分为三类：

1. 独立数据库
2. 同一数据库下，不同数据表
3. 同一数据库下，同一数据表，基于tenant_id区分

一般我们设计 SaaS 架构时，会选择前两种，但不管是前两者中的哪一种，我们都有一个必须解决的问题——**如何动态切换数据源**

### 独立数据库

> 一个租户一个数据库，成本最高，安全最好
>
> TODO 等后续扩展分布式应用后，再实现

优点：

1. 为不同的租户提供了不同的数据库，有利于简化数据模型的设计和扩展，满足不同租户的独特需求；
2. 如果出现故障，数据恢复方便。
3. 不同租户之间的性能可以按需调配

缺点：

1. 维护成本和维护成本较高
2. 跨租户统计复杂

### 同数据库下，不同表

> 所有租户一个数据库，但一个租户一个表
>
> TODO 等后续扩展分布式应用后，再实现

优点：

1. 为安全性较高的租户提供了一定程度的数据结构隔离，但并没有完全隔离
2. 一个数据库可以支持更多的租户

缺点：

1. 如果故障，回复困难，可能存在牵扯不同租户的数据
2. 跨租户统计复杂

### 同库同表，基于tenant_id

> 所有租户一个数据库，且同一个表。但在表中基于tenant_id区分租户。
>
> 这是共享程度最高，但隔离即便最低的模式
>
> TODO 等后续扩展分布式应用后，再实现

优点：

1. 共享程度高，但安全性低
2. 维护和购置成本最低，允许每个数据库支持的租户数量最多。 

缺点：

1. 隔离级别最低，安全性最低，需要在设计开发时加大对安全的开发量；
2. 数据好迁移
3. 数据备份和恢复最困难，需要逐表逐条备份和还原



