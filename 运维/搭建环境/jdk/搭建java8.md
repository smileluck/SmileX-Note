[toc]

---
# 说明

## 支持系统

- 安装方式
	- 支持 Ubuntu 18.04
	- 支持 Ubuntu 16.04
	- 支持 Centos 7

# 安装方式1


1. 下载jdk，二进制压缩包.tar.gz结尾的。

2. 上传jdk到服务器上，用户目录下。例如/home/ubuntu。
3. 解压tar文件
```shell
cd /usr/local
sudo mkdir java
cd java
sudo tar -zxvf /home/ubuntu/jdk-8u261-linux-x64.tar.gz -C /usr/local/java
```

4. 配置环境变量。用VIM来打开环境变量配置文件
```shell
vim /etc/profile
```

5. 在文件末尾添加如下代码
```sh
export JAVA_HOME=/usr/local/java/jdk1.8.0_261
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
6. 保存文件， 使该文件生效
```shell
source /etc/profile
```

7. 验证java安装是否成功

```shell
 java -version
````

# 踩坑记录

## sudo: java: command not found

当我们需要用 sudo 来执行java命令或者 .sh文件时，会提示指令未发现。明明我们使用在环境变量里配置的，应该怎么处理呢？

实际应该修改 **/etc/sudoers** 文件

1. vim编辑/etc/sudoers文件

```shell
vim /etc/sudoers
```

2. 在 Defaults    secure_path 的后面增加 :/usr/local/java/jdk1.8.0_261/bin

```sh
# 之前
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

# 之后
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin:/usr/local/java/jdk1.8.0_261/bin"
```

3. 再试验一下，输出版本即可。

```shell
sudo java -version
```

