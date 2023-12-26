[toc]

---

# 连接mysql后，无法执行多条记录

## 问题描述

SQL 如下

```sql
create database test_char;

use test_char;

create table user(id int,name varchar(20));

insert into user(id,name) value(1,"张三");
insert into user(id,name) value(2,"李四");

select * from `user`;	
```

异常信息：

```shell
[1064] You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'use test_char;

create table t_user (id int,name varchar(20));

insert into ' at line 3
```

## 解决方法

1. 确保每行SQL语句后面有加 `;`
2. 在数据库连接串上加上 `allowMultiQueries`  属性为 `true`
3. 对于 `Dbeaver`  工具连接的，设置地址为：编辑连接->连接属性->驱动属性