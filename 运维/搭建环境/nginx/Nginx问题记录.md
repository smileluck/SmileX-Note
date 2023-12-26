[toc]

---

# Nginx 报错 Permission denied 没有权限

1. 打开 `nginx.conf`

```shell
cd /etc/nginx

sudo vim nginx.conf
```

2. 修改 `nginx.conf` 第一行

```json
user ubuntu; # 改成自己的用户名
```

3. 保存并重载配置

```shell
sudo nginx -s reload
```

# nginx: [error] open() "/run/nginx.pid" failed

1. 查看进程列表

```shell
ps -ef |grep nginx
```

2. 杀死进程

```shell
sudo kill -9 pid
```

3. 重新启动 nginx

```shell
sudo nginx
```

