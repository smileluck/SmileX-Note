[toc]

---

# 前言

`nacos` 中主要使用两种数据源：内置和外置。

有些时候我们使用内置数据源时，并不方便做数据查看和统一配置。所以我们会需要使用外置数据源进行配置

# 外置数据源 

> 目前仅支持 Mysql

1. 安装数据库，版本要求：5.6.5+
2. 初始化mysql数据库，数据库初始化文件：`mysql-schema.sql`
3. 3.修改`conf/application.properties`文件，增加支持mysql数据源配置（目前只支持mysql），添加mysql数据源的url、用户名和密码。

```properties
### Deprecated configuration property, it is recommended to use `spring.sql.init.platform` replaced.
#spring.datasource.platform=mysql
spring.sql.init.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
db.url.0=jdbc:mysql://127.0.0.1:3306/smilex-nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=root
db.password.0=root
```

