[toc]

---

# 说明

## 支持系统

- 方法1
  - ubuntu 18.04
  - ubuntu 16.04

# 方法一：Apt-get

> 使用apt-get工具安装

## 安装步骤

### 安装软件

```shell
sudo apt-get install -y redis-server
```

### 配置密码(可选)

```shell
sudo vim /etc/redis/redis/redis.conf
```

找到 requirepass ，关闭注释并将后面的字符串修改成想要设置的密码。例如

```shell
requirepass 123456 #将密码设置为123456
```

## 服务操作

如果是用apt-get或者yum install安装的redis，可以直接通过下面的命令停止/启动/重启redis

```shell
/etc/init.d/redis-server stop
/etc/init.d/redis-server start
/etc/init.d/redis-server restart
```

# 方法二：源码安装

## 安装步骤

### 官网下载源码

官网：[下载地址](https://redis.io/download/)

然后在Ubuntu上操作,源码安装redis

1. 下载
```shell
wget https://download.redis.io/releases/redis-6.0.9.tar.gz
```
2. 解压到了/user/local目录下
```shell
tar -xvf redis-6.0.9.tar.gz -C /usr/local/
```
3. 进入你移动的目录
```shell
cd /usr/local/redis-6.0.9
```
4. 编译Redis
```shell
sudo make
```
5. 测试编译是否成功，可略过(这一步时间会比较长，耗时5分钟左右)
```shell
sudo make test
```
6. 安装
```shell
sudo make install
```