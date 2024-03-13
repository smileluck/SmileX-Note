[toc]

---

# for update

`for update` 会给读操作上写锁（排他锁）。因为是写锁，如果上锁时，有另一事务对此数据加写锁，那么当前事务会等待写锁释放（提交或者回滚）后拿到锁，在执行操作。

常用场景：

- 用户金额
- 商品余量

## 基本用法

在查询语句里面加上 `for update`，例如：

```java
select * from table where id = 1 for update;
```

不同数据库的扩展写法：

- Oracle

  - select * from table where id = 1 for update no wait; 不等待，获取不到则抛出异常返回
  - select * from table where id = 1 for update wait 3; 等待3s，获取不到则抛出异常返回

- Mysql

  - ```mysql
    set session innodb_lock_wait_timeout=3;
    select * from kb_power_compere where user_id = #{userId}  and del_flag = 0 for update;
    ```

    设置当前会话的锁等待时间，当超过的等待时间后，会抛出异常。
    
  - mybatis 下，需要这么写，借助属性timeout，单位s
  
    ```xml
    <select id="selectForUpdate" timeout="20">
    select * from kb_power_compere where user_id = #{userId}  and del_flag = 0 for update;
    </select>
    ```
    
    



## 注意事项

for update是根据where的索引情况进行加锁，根据具体情况，可能会产生：行级锁、间隙锁、表级锁。

1. 当查询语句走主键/唯一键索引，且数据全部命中，锁住单行。（即使是范围查询，比如 where id in (1,2,3)，如果都存在，也是只锁1,3,5三行）。
2. 当查询语句走主键/唯一键索引，但数据部分命中，或都不命中；或走非唯一索引：用间隙锁，锁住区间行。
3. 当查询语句不走索引，会用间隙锁把整张表锁住（但其实并不是表锁），因此要尽量避免索引失效的场景。



# 参考

-  https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html#innodb-gap-locks 