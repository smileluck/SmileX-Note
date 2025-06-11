

# 配置缓存位置

## 全局配置
1. 方式一：环境变量 `GRADLE_USER_HOME`
```shell
echo $env:GRADLE_USER_HOME  # PowerShell
echo %GRADLE_USER_HOME%    # CMD
```

2. 方式二：修改初始化脚本文件 `%USERPROFILE%\.gradle\init.gradle`
```gradle
gradle.settingsEvaluator.doSetSettings { settings ->
    settings.buildCache {
        local {
            directory = "/path/to/your/gradle/cache"
        }
    }
}
```

3. 方式三：修改 `%USERPROFILE%\.gradle\gradle.properties`
```properties
# 指定缓存目录
org.gradle.caching=true
org.gradle.cache.dir=D:/cache/gradle
```


## 具体项目修改（类似flutter）

1. 方式一： 修改 `gradlew.bat` 文件

> 不知道为啥flutter必须得这种方式才行

```bat
@rem Add default JVM options here. You can also use JAVA_OPTS and GRADLE_OPTS to pass JVM options to this script.
set DEFAULT_JVM_OPTS=
@rem add this sentence to enable gradle cache
set GRADLE_OPTS=-Dgradle.user.home=D:\cache\gradle
```