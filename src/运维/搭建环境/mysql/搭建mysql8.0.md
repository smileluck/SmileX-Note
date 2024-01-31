[toc]

---

# 说明



支持系统: 

- ubuntu 18.04





# 方法一

1. 切换路径，下载安装包

```shell
cd /usr/local
mkdir mysql-8.0
cd /mysql-8.0
wget -c https://dev.mysql.com/get/mysql-apt-config_0.8.10-1_all.deb
```

2. 安装程序

```shell
sudo dpkg -i mysql-apt-config_0.8.10-1_all.deb
``` 

3. 下载最新的软件包信息 

```shell
sudo apt update	
```

4. 如果出现异常 仓库 “http://repo.mysql.com/apt/[ubuntu](https://so.csdn.net/so/search?q=ubuntu&spm=1001.2101.3001.7020) bionic InRelease” 没有数字签名 。请看踩坑记录
5. 查看升级

```shell
apt list --upgradable
```

6.  安装mysql8.0 

```shell
sudo apt install mysql-server
```

7.  验证安装 

```shell
mysql -uroot -p
```

8. 设置远程访问权限

```shell
SHOW DATABASES;
USE mysql;
SELECT 'host' FROM user WHERE user='root'; #查看user表的host值
UPDATE user SET host = '%' WHERE user ='root'; #修改host值
flush privileges; #刷新MySQL的系统权限相关表
```

9. 修改绑定IP

```shell
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

# 更改bind-address
bind-address = 0.0.0.0
```

# 方法二（win）

> 这里使用版本：mysql-8.0.33-winx64

1. 登录官网下载[免安装版本](https://dev.mysql.com/downloads/file/?id=518220)。

2. 解压并进入软件根目录。

3. 新建`my.ini`文件

   ```ini
   [mysqld]
   # 设置3307端口
   port=3307
   # 设置mysql的安装目录，一定要与上面的安装路径保持一致
   basedir=D:\dev-tools\mysql-8.0.33-winx64
   # 设置mysql数据库的数据的存放目录
   datadir=D:\dev-tools\mysql-8.0.33-winx64\data
   # 允许最大连接数
   max_connections=200
   # 允许连接失败的次数。
   max_connect_errors=10
   # 服务端使用的字符集默认为utf8mb4
   character-set-server=utf8mb4
   # 创建新表时将使用的默认存储引擎
   default-storage-engine=INNODB
   # 默认使用“mysql_native_password”插件认证
   #mysql_native_password
   default_authentication_plugin=mysql_native_password
   [mysql]
   # 设置mysql客户端默认字符集
   default-character-set=utf8mb4
   [client]
   # 设置mysql客户端连接服务端时默认使用的端口 可以根据实际情况进行修改
   port=3307
   default-character-set=utf8mb4
   ```

4. 进入 \bin 目录，执行以下指令。

   - --defaults-file。指定默认文件

   ```shell
   mysqld --defaults-file=C:\Softwares\mysql-8.0.12-winx64\my.ini --initialize --console ##使用管理员的方式打开
   ```

   输出结果：

   ```shell
   2023-05-05T07:22:04.819909Z 0 [Warning] [MY-010918] [Server] 'default_authentication_plugin' is deprecated and will be removed in a future release. Please use authentication_policy instead.
   2023-05-05T07:22:04.819935Z 0 [System] [MY-013169] [Server] D:\dev-tools\mysql-8.0.33-winx64\bin\mysqld.exe (mysqld 8.0.33) initializing of server in progress as process 920
   2023-05-05T07:22:04.840333Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
   2023-05-05T07:22:06.123640Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
   2023-05-05T07:22:07.586539Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: b*qMh0L>2-;C
   ```

   最后面的就是临时密码。这里看出是： `b*qMh0L>2-;C`

5. 执行数据库初始化。提示 **Service successfully installed** 表示成功。

```shell
mysqld --install MySQL-8.0.33 # MySQL-8.0.12为服务名称，默认为Mysql
```

6. 开启/关闭 MySql

```shell
net start MySQL-8.0.33
net stop MySQL-8.0.33
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
      D:\dev-tools\mysql-8.0.33-winx64\bin;
      ```

   2. 自定义脚本启动

      ```shell
      @echo off
      
      rem 设置 MySQL 用户名和密码
      set MYSQL_USERNAME=root
      set MYSQL_PASSWORD=123456
      
      rem 设置 MySQL 端口号，默认为 3306
      set MYSQL_PORT=3307
      
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

      


# 踩坑记录

##  仓库 “http://repo.mysql.com/apt/[ubuntu](https://so.csdn.net/so/search?q=ubuntu&spm=1001.2101.3001.7020) bionic InRelease” 没有数字签名 

执行 `apt update`时，报出异常。

![image](搭建mysql8.0.assets/1669883831227.jpg)

1. 查看签名列表

```shell
apt-key list
```

2.  删除过期签名 

```shell
sudo apt-key del dsa1024
```

3.  重新添加新的签名。这里的key为 上面红色框中的 key 

```shell
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 467B942D3A79BD29
```

## 关于groupby 的sqlmode配置问题

1. 进入mysqld配置文件目录

```shell
cd /etc/mysql/mysql.conf.d
```

2. 增加sql_mode

```shell
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION 

sql_mode=NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
```

# 启动服务

1. 启动服务

```shell
cd /usr/bin
sudo mysqld restart --user=root
```

