[toc]

---

# 我的部署

```shell
# 进入目录
cd /usr/local/serve

# 创建文件夹jar，用来放Jar文件
mkdir jar

# 移动文件，从home/ubuntu移动该目录下
mv ~/test.jar /usr/local/serve/jar

# 赋予文件权限
sudo chmod u+x test-start.sh 
sudo chmod u+x test.jar
sudo chmod 777 -R /usr/local/serve # 简单粗暴

# 执行test-start.sh
./test-start.sh
```



# 常用指令

1. 查看进程

```shell
# 查看所有进程
ps -ef 

# 查看所有java进程
ps -ef |grep java
```

2. 杀死java进程。根据PID杀死进程

```shell
kill -9 [pid]
```

3. 查看文件日志。假设日志文件名为nohup.out

```shell
# 查看最新的200条日志
tail -fn 200 nohup.out

# 查找文件中间内容
cat nohup.out | head -n 20958238 | tail -n +20957300

# 查找文件内容匹配的行数
grep "mac-->" -nR nohup.out
```

4. 远程调试。记得开放端口

```shell
java -Duser.timezone=GMT+08 -Xdebug -Xrunjdwp:transport=dt_socket,address=5555,server=y,suspend=y -jar test.jar
```

5. 启动服务。

- 常用参数
  - -Dspring.profiles.active=test。指定profile为test
  - -Duser.timezone=GMT+08。设置时区
  - -server。服务器模式。默认为混合模式
  - --logging.file=/var/log/test.log。设置日志文件位置

```shell
# 控制台前台启动
java -Duser.timezone=GMT+08 -jar test.jar

# 后台启动
nohup java -Duser.timezone=GMT+08 -jar test.jar &

# 后台启动并指定输出日志文件
java -server -Xms512m -Xmx512m  -Duser.timezone=GMT+08 -jar test.jar > test.out 2>&1 &
```



# 脚本启动

## 通过shell脚本
1. 创建test-start.sh文件，内容如下。
```sh
#!/bin/bash
PID=$(ps -ef|grep ytqms-wx-1.0-SNAPSHOT.jar| grep -v grep | grep -v tail | awk '{printf $2}')

if [ $? -eq 0 ]; then
    echo "查询进程ID为 : $PID"
else
    echo "脚本执行失败,退出"
	exit
fi

if [ ! -n "$PID" ] || [ ! $PID ] || [ "$PID" = "" ]; then
	echo "未查询到PID,直接启动项目"
	java -server -Xms512m -Xmx512m -Dspring.profiles.active=prod -Duser.timezone=GMT+08 -jar ytqms-wx-1.0-SNAPSHOT.jar > ytqms-wx-1.0-SNAPSHOT.out 2>&1 &
else
	kill -9 ${PID}

	if [ $? -eq 0 ];then
		echo "杀死进程成功,启动项目"
		java -server -Xms512m -Xmx512m -Dspring.profiles.active=prod -Duser.timezone=GMT+08 -jar ytqms-wx-1.0-SNAPSHOT.jar > ytqms-wx-1.0-SNAPSHOT.out 2>&1 &
	else
		echo "杀死进程失败"
	fi
fi
```
2. 赋予文件权限

```shell
sudo chmod u+x test-start.sh
```

3. 运行文件
```shell
sudo ./test-start.sh
```

## 配置service

1. 进入/etc/systemd/stystem，新建cloud-test.service文件

```shell
cd /etc/systemd/system

vim cloud-test.service
```

2. 文件内容如下

```sh
[Unit]
Description=cloud-test-service server daemon
After=network-online.target
Wants=network-online.target

[Service]
User=root
Group=root
ExecStart=/bin/sh -c 'exec java -jar /home/ubuntu/serve/jar/cloud-test.jar --logging.file=/var/log/cloud-test.log'
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failur
RestartSec=60s

[Install]
```

3. 保存文件并注册服务。更多指令可查看systemctl.md文件。

```shell
# 重新加载所有被修改过的服务配置，否则配置不会生效
sudo systemctl daemon-reload

# 系统开机时自动启动指定unit，前提是配置文件中有相关配置
sudo systemctl enable test.service

# 启动服务
sudo systemctl start test.service

# 停止服务
sudo systemctl stop test.service

# 重启服务
sudo systemctl restart test.service

# 重新加载配置
sudo systemctl reload test.service
```

