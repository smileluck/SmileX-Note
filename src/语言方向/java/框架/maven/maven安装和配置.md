[toc]

---

# maven 安装和配置
> 这里使用版本是 v3.9.6。   
> 下载地址：https://maven.apache.org/download.cgi

1. 解压到本地文件夹下
2. 打开 `config\settings.xml`
3. 配置本地存储位置
    ```xml
    <localRepository>D:/dev/repositories/maven</localRepository>
    ```
4. 配置阿里云镜像仓库
   ```xml
    <mirror>
      <id>aliyunmaven</id>
      <mirrorOf>*</mirrorOf>
      <name>阿里云公共仓库</name>
      <url>https://maven.aliyun.com/repository/public</url>
    </mirror>
   ```
5. 私有仓库配置（带密码）
    ```xml
	<servers>
		<server>
			<id>nexus-releases</id>
			<username>admin</username>
			<password>xxx</password>
		</server>
	</servers>
    ```
6. 配置java版本(以1.8为例子)
    ```xml
    <profile>
        <id>JDK-1.8</id>
        <activation>
            <activeByDefault>true</activeByDefault>
            <jdk>1.8</jdk>
        </activation>
        <properties>
            <maven.compiler.source>1.8</maven.compiler.source>
            <maven.compiler.target>1.8</maven.compiler.target>
            <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
        </properties>
    </profile>
    ```
7. 配置环境变量
   - MAVEN_HOME。D:\dev\apache-maven-3.9.6
   - PATH。添加 %MAVEN_HOME%\bin

# 多模块

## packaging	

在当前分布式和微服务日益发展壮大的背景下，我们需要更了解关于maven配置的细节，以及多模块下需要注意的地方。

目前 packaging 具有以下3中配置：

1.  <packaging>pom</packaging>
2. <packaging>jar</packaging>
3. <packaging>war</packaging> 



### pom

在多模块项目中，我们会把某一类模块（例如 common 模块）聚合所有通用的模块，例如log、upload等，但时本身common模块不参与打包使用，只是作为一个聚合管理，那么这时我们可以将其设置为

```xml
<packagin>pom</packagin>
```

为什么一定要有一个父类的POM呢？大致有以下几点好处：

1. 可以通过 `Modules` 标签管理子模块的编译顺序（Maven 本身采用最短路径原则）。
2. 将子模块的公共依赖包和版本交由父模块进行管理
3. groupId\artifactid\version可以从父类获取，减少子模块POM的复杂性。

### jar

当没有配置 packaging 时，默认值为jar，即采用jar方式打包.

这种打包方式意味着在maven build时会将这个项目中的所有java文件都进行编译形成.class文件，且按照原来的java文件层级结构在target目录下放置，最终压缩为一个jar文件。

### war

war包与jar包非常相似，同样是编译后的.class文件按层级结构形成文件树后打包形成的压缩包。不同的是，它会将项目中依赖的所有jar包都放在WEB-INF/lib这个文件夹下。

可想而知，war包非常适合部署时使用，不再需要下载其他的依赖包，能够使用户拿到war包直接使用，因此它经常使用于[微服务](https://so.csdn.net/so/search?q=微服务&spm=1001.2101.3001.7020)项目群中的入口项目的pom配置中。

# 异常记录

##  'packaging' with value 'jar' is invalid. aggregator projects require 'pom'

需要将父模块的packaging设置为pom即可解决