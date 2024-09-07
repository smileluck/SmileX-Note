[TOC]

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

# 方法二（win）

> 这里使用版本：mysql-5.7.44-winx64

1. 登录官网下载[免安装版本](https://dev.mysql.com/downloads/mysql/)。

2. 解压并进入软件根目录。

3. 新建`my.ini`文件
   
   ```ini
   [mysqld]
   # 设置3306端口
   port=3306
   # 设置mysql的安装目录，一定要与上面的安装路径保持一致
   basedir=D:\dev-tools\mysql-5.7.44-winx64
   # 设置mysql数据库的数据的存放目录
   datadir=D:\dev-tools\mysql-5.7.44-winx64\data
   # 允许最大连接数
   max_connections=200
   # 允许连接失败的次数。
   max_connect_errors=10
   # 服务端使用的字符集默认为utf8mb4
   character-set-server=utf8mb4
   # 创建新表时将使用的默认存储引擎
   default-storage-engine=INNODB
   
   [mysql]
   # 设置mysql客户端默认字符集
   default-character-set=utf8mb4
   [client]
   # 设置mysql客户端连接服务端时默认使用的端口 可以根据实际情况进行修改
   port=3306
   default-character-set=utf8mb4
   ```

4. 进入 \bin 目录，执行以下指令。
   
   - --defaults-file。指定默认文件
   
   ```shell
   mysqld --defaults-file=C:\Softwares\mysql-5.7.44-winx64\my.ini --initialize --console ##使用管理员的方式打开
   ```
   
   输出结果：
   
   ```shell
   2023-05-05T07:22:04.819909Z 0 [Warning] [MY-010918] [Server] 'default_authentication_plugin' is deprecated and will be removed in a future release. Please use authentication_policy instead.
   2023-05-05T07:22:04.819935Z 0 [System] [MY-013169] [Server] D:\dev-tools\mysql-5.7.44-winx64\bin\mysqld.exe (mysqld 8.0.33) initializing of server in progress as process 920
   2023-05-05T07:22:04.840333Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
   2023-05-05T07:22:06.123640Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
   2023-05-05T07:22:07.586539Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: b*qMh0L>2-;C
   ```
   
   最后面的就是临时密码。这里看出是： `b*qMh0L>2-;C`

5. 执行数据库初始化。提示 **Service successfully installed** 表示成功。

```shell
mysqld --install mysql-5.7.44 # mysql-5.7.44为服务名称，默认为Mysql
```

6. 开启/关闭 MySql

```shell
net start mysql-5.7.44
net stop mysql-5.7.44
```

7. 登录mysql 修改密码

```shell
#mysql -u root -p初始化的密码    #登录mysql
mysql -u root -p'b*qMh0L>2-;C' 

#ALTER USER 'root'@'localhost' IDENTIFIED BY '新密码';  # 修改密码
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';  # 修改密码
```

8. 启动方式
   
   1. 加入到环境变量 Path 中启动：将**安装地址\bin**追加到Path最后面
      
      ```
      D:\dev-tools\mysql-5.7.44-winx64\bin;
      ```
   
   2. 自定义脚本启动
      
      ```shell
      @echo off
      
      rem 设置 MySQL 用户名和密码
      set MYSQL_USERNAME=root
      set MYSQL_PASSWORD=123456
      
      rem 设置 MySQL 端口号，默认为 3306
      set MYSQL_PORT=3306
      
      rem 设置 MySQL 根目录路径，根据实际情况调整路径
      set MYSQL_HOME=%cd%
      
      #rem 设置 SQL 初始化文件的路径，用于创建初始化库等
      #set SQL_INIT_FILE=%~dp0init.sql
      
      rem 初始化 数据存储目录
      "%MYSQL_HOME%\bin\mysqld" --initialize-insecure
      
      rem 启动 MySQL 服务
      echo Starting MySQL server...
      "%MYSQL_HOME%\bin\mysqld" --port=%MYSQL_PORT% --console --user=%MYSQL_USERNAME% --bind-address=127.0.0.1 --datadir="%MYSQL_HOME%\data"  --shared-memory   
      #--init-file="%SQL_INIT_FILE%"
      
      echo 
      if %errorlevel% neq 0 (
          echo Failed to start MySQL server.
          exit /b %errorlevel%
      )
      
      echo MySQL server started successfully and new database created.
      exit /b 0
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
