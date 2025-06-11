[toc]

---



# 前言

在Maven多模块的时候，管理依赖关系是非常重要的，各种依赖包冲突，查询问题起来非常复杂，于是就用到了<dependencyManagement>，

# 示例说明
在父模块中：
```pom
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.44</version>
            </dependency>
           
        </dependencies>
</dependencyManagement>
```

那么在子模块中只需要<groupId>和<artifactId>即可，如：

```pom
 <dependencies>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
 </dependencies>
```

## 说明
使用dependencyManagement可以统一管理项目的版本号，确保应用的各个项目的依赖和版本一致，不用每个模块项目都弄一个版本号，不利于管理，当需要变更版本号的时候只需要在父类容器里更新，不需要任何一个子项目的修改；如果某个子项目需要另外一个特殊的版本号时，只需要在自己的模块dependencies中声明一个版本号即可。子类就会使用子类声明的版本号，不继承于父类版本号。

# 与dependencies区别

1. Dependencies相对于dependencyManagement，所有生命在dependencies里的依赖都会自动引入，并默认被所有的子项目继承。
2. dependencyManagement里只是声明依赖，并不自动实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom;另外如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。

# 使用案例
spring-boot-dependencies 和 spring-boot-starter-parent。

我们在日常开发中经常使用的spring-boot-starter-parent（继承了spring-boot-dependencies）里面其实就提供了版本控制，所以我们在项目的pom文件中的 dependencies 中引入一些库时，可以不需要填入版本，会自动去使用 spring-boot-dependencies 中规定的版本。