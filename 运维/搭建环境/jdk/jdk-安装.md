[toc]

---

# 安装JDK

JDK通常是从[ Oracle官网](https://www.oracle.com/)下载， 打开页面翻到底部，找 Java for Developers 或 者 Developers , 进入 [Java 相应的页面](https://www.oracle.com/java/technologies/) 或者 [Java SE 相应的页面](https://www.oracle.com/java/technologies/java-se-glance.html), 查找 Download, 接受许可协议,下载对应的x64版本即可。


> 注意：从Oracle官方安装JDK需要注册和登录Oracle账号。现在流行将下载链接放 到页面底部，很多工具都这样。**当前推荐下载 JDK8。 今后JDK11将会成为主流 版本，因为Java11是LTS长期支持版本（2019年10月15日发布的JDK11.0.5已经 被官方标记为LTS版本）**，但可能还需要一些时间才会普及，而且JDK11的文件目 录结构与之前不同, 很多工具可能不兼容其JDK文件的目录结构。

有的操作系统提供了自动安装工具，直接使用也可以，比如 yum, brew, apt 等等。例 如在MacBook上，执行：
> brew cask install java8

而使用如下命令，会默认安装最新的JDK13：

> brew cask install java

# 设置环境变量
如果找不到命令，需要设置环境变量： ==JAVA_HOME== 和 ==PATH== 。

> JAVA_HOME 环境变量表示JDK的安装目录，通过修改 JAVA_HOME ，可以快速切换JDK版本。很多工具依赖此环境变量。

> 另外, 建议不要设置 CLASS_PATH 环境变量，新手没必要设置，容易造成一些困 扰

Windows系统, 系统属性 - 高级 - 设置系统环境变量。 如果没权限也可以只设置用户 环境变量。 

Linux和MacOSX系统, 需要配置脚本。 例如:
> cat ~/.bash_profile

```shell
# JAVA ENV
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_162.jdk/Contents/Home
export PATH=$PATH:$JAVA_HOME/bin
```

让环境配置立即生效:
> source ~/.bash_profile

查看环境变量:
```
echo $PATH 
echo $JAVA_HOME
```

一般来说， .bash_profile 之类的脚本只用于设置环境变量。 不设置随机器自启动的程序。

如果不知道自动安装/别人安装的JDK在哪个目录怎么办?

查找的方式很多，比如，可以使用 ==which ， whereis ， ls ‐l==跟踪软连接, 或者 ==find --命令全局查找(可能需要sudo权限), 例如:

```
jps ‐v 
whereis javac 
ls ‐l /usr/bin/javac
find / ‐name javac
```

找到满足 $JAVA_HOME/bin/javac 的路径即可

Windows系统，安装在哪就是哪，默认在 C:\Program Files (x86)\Java 下。通 过任务管理器也可以查看某个程序的路径，注意 JAVA_HOME 不可能是 C:\Windows\System32 目录。

然后我们就可以在JDK安装路径下看到很多JVM工具

# 验证JDK安装完成
安装完成后，Java环境一般来说就可以使用了。 验证的脚本命令为:
> java ‐version

可以看到输出类似于以下内容，既证明成功完成安装：

> java version "1.8.0_65" Java(TM) SE Runtime Environment (build 1.8.0_65­b17) Java HotSpot(TM) 64­Bit Server VM (build 25.65­b01, mixed mode)

然后我们就可以写个最简单的java程序了，新建一个文本文件，输入以下内容：
```java
 public class Hello { 
    public static void main(String[] args){
        System.out.println("Hello, JVM!"); 
    } 
 }
```

 然后把文件名改成 Hello.java ，在命令行下执行：

 > javac Hello.java

 然后使用如下命令运行它：

 > java Hello 