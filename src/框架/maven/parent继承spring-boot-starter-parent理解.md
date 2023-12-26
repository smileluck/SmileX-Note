[toc]

---



# 前言

在很多的springboot项目中，我们都能看到pom中，有类似这样的一段代码：
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.5.RELEASE</version>
</parent>
```
因为我这里用的版本是2.3.5.RELEASE，可能会存在一定的出入。

那么这一段有什么作用呢？让我们通过查看对应版本的pom文件内容，分析一下看看。 

# 功能
这里因为篇幅原因，我只截取关键的代码。

```xml
<!-- spring-boot-starter-parent-2.3.5.RELEASE.pom -->
 <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>2.3.5.RELEASE</version>
  </parent>

<properties>
    <java.version>1.8</java.version>
    <resource.delimiter>@</resource.delimiter>
    <maven.compiler.source>${java.version}</maven.compiler.source>
    <maven.compiler.target>${java.version}</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
</properties>

<resources>
  <resource>
    <directory>${basedir}/src/main/resources</directory>
    <filtering>true</filtering>
    <includes>
      <include>**/application*.yml</include>
      <include>**/application*.yaml</include>
      <include>**/application*.properties</include>
    </includes>
  </resource>
  <resource>
    <directory>${basedir}/src/main/resources</directory>
    <excludes>
      <exclude>**/application*.yml</exclude>
      <exclude>**/application*.yaml</exclude>
      <exclude>**/application*.properties</exclude>
    </excludes>
  </resource>
</resources>

<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <configuration>
    <parameters>true</parameters>
  </configuration>
</plugin>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-failsafe-plugin</artifactId>
  <executions>
    <execution>
      <goals>
        <goal>integration-test</goal>
        <goal>verify</goal>
      </goals>
    </execution>
  </executions>
  <configuration>
    <classesDirectory>${project.build.outputDirectory}</classesDirectory>
  </configuration>
</plugin>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <configuration>
    <archive>
      <manifest>
        <mainClass>${start-class}</mainClass>
        <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
      </manifest>
    </archive>
  </configuration>
</plugin>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-war-plugin</artifactId>
  <configuration>
    <archive>
      <manifest>
        <mainClass>${start-class}</mainClass>
        <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
      </manifest>
    </archive>
  </configuration>
</plugin>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-resources-plugin</artifactId>
  <configuration>
    <delimiters>
      <delimiter>${resource.delimiter}</delimiter>
    </delimiters>
    <useDefaultDelimiters>false</useDefaultDelimiters>
  </configuration>
</plugin>
```

1. 指定jdk版本为1.8
2. 指定字符集为UTF-8
3. 继承了spring-boot-dependencies.2.3.5.RELEASE。
4. 针对以yml、yaml、properties结尾的文件资源过滤，包括以profile定义的不同环境的文件。
5. 自动化的插件配置，
    - 资源插件：maven-resources-plugin（处理资源文件到输出目录）
    - 测试插件：maven-failsafe-plugin
    - 打包插件：
        - maven-jar-plugin（打包jar）
        - maven-war-plugin(打包war)
6. 依赖版本的管理。是由spring官方测试过后，较为兼容的版本，可以在spring-boot-dependencies.2.3.5.RELEASE中查看到。

spring-boot-dependencies中是通过 dependencyManagement 和 pluginManagement 对版本进行管理的。
对 dependencies 和 dependencyManagement 差别可以看一下[这个文章](https://blog.csdn.net/dubismile/article/details/121427625)

# 不继承spring-boot-parent
那么不使用parent会有什么后果呢？显而易见的，就是上面的parent提供的功能都要自己实现

## 基础配置
```xml

<properties>
    <java.version>1.8</java.version>
    <resource.delimiter>@</resource.delimiter>
    <maven.compiler.source>${java.version}</maven.compiler.source>
    <maven.compiler.target>${java.version}</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
</properties>
```
需要自己手动指定字符集和编译版本。

## 版本依赖
因为没有地方对依赖版本进行了控制，所以我们所有引入的依赖都需要自己填入版本。
```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.3.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
            <version>2.3.5.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
            <version>2.3.5.RELEASE</version>
        </dependency>
</dependencies>
```
这时项目在本地能够正常运行，因为依赖的配置都能获取到，但是无法打包成jar或者war包。

## 插件配置
```xml
 <plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-jar-plugin</artifactId>
  <configuration>
    <archive>
      <manifest>
        <mainClass>${start-class}</mainClass>
        <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
      </manifest>
    </archive>
  </configuration>
</plugin>
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-war-plugin</artifactId>
  <configuration>
    <archive>
      <manifest>
        <mainClass>${start-class}</mainClass>
        <addDefaultImplementationEntries>true</addDefaultImplementationEntries>
      </manifest>
    </archive>
  </configuration>
</plugin>
```
我们就需要自己配置对应的插件和版本，像是打包war包所需要的web.xml文件，指定主类等，都需要手动配置。当然我们也可以借助其它工具来打包。

# 直接继承spring-boot-dependencies
实际上spring-boot-dependencies主要的功能就是提供了一个对依赖版本的管理。让我们对版本的控制可以简化，因为是由spring官方测试过的稳定兼容版本。然后我们自己还是需要做一个类似 spring-boot-parent 的pom文件，因为这是我们实际依赖的项目插件和配置。

说简单点了，实际就是dependencyManagement和dependencies的关系。

# 总结
这主要是springboot的主要理念：起步依赖、简化开发。我们只需要简单的通过继承spring-boot-parent，就能快速的解决编译、资源文件输出、配置文件、打包、版本依赖等管理，而不需要繁杂的配置。