[toc]

---

# 问题记录

## SQL错误【1290】【HY000】

> The MySQL server is running with the --secure-file-priv option so it cannot execute this statement

引发问题`SQL`。引发原因是由于 `into outfile` 出发了mysql的安全配置权限问题。

```sql
select * from temp into outfile 'D://1.txt';
```

查看当前的安全配置。

```sql
show variables like "%secure%";
```

配置方法，打开配置文件 `my.ini`

```ini
[mysqld]
# 配置安全配置导出路径前缀
secure-file-priv = D:\dev-tools\mysql-8.0.33-winx64\output
```

重启`mysql` 服务即可。

