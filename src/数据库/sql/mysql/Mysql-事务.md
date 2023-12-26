[toc]

---

# 前言

## 事务是什么？

 事务是由一个或多个sql语句组成一个最小的不可再分的工作单元。里面的内容要么都执行成功，要么都不成功。 

## 事务的特性(ACID)

- 原子性（atomicity）。事务是一个不可分割的工作单元，要么全部提交，要么全部失败回滚。
- 一致性（consistency）。一致性指事务执行前后，数据从一个合法性状态变换到另一个合法性状态。例如要满足存在的约束，满足数据的一致性等、
- 隔离性（isolation）指一个事务的执行不能被其他事务干扰。
- 持久性（durability）一个事务一旦提交成功，它对数据库中数据的改变将是永久性的，接下来的其他操作或故障不应对其有任何影响。

## 存储引擎支持情况

可以通过如下指令查询存储引擎对事务的支持情况。

```mysql
show engines; 
```
你就可以得到类似如下的表格。可以看出在 `Mysql5.7 ` 中只有 `InnoDB` 是支持事务的。

| Engine    | Support    | Comment | Transactions | XA | Savepoints |
| ---- | ---- | ---- | ---- | ---- | ---- |
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
