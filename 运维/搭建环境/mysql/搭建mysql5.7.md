[toc]

---

# 说明



支持系统: 

- ubuntu 18.04



# 方法一

```shell
sudo apt-get install mysql-server 
sudo apt-get install mysql-client 
sudo apt-get install libmysqlclient-dev
```







# 彻底删除mysql安装记录

```shell
# 首先用以下命令查看自己的mysql有哪些依赖包

dpkg --list | grep mysql

# 先依次执行以下命令

sudo apt-get remove mysql-common

sudo apt-get autoremove --purge mysql-server-5.0    # 卸载 MySQL 5.x 使用,  非5.x版本可跳过该步骤

sudo apt-get autoremove --purge mysql-server

# 然后再用 dpkg --list | grep mysql 查看一下依赖包

# 最后用下面命令清除残留数据

dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P

# 查看从MySQL APT安装的软件列表, 执行后没有显示列表, 证明MySQL服务已完全卸载

dpkg -l | grep mysql | grep i

```

