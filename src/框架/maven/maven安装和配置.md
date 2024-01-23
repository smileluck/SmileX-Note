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