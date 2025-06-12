- [Flutter 连接 https://pub.dev 报错解决](#flutter-连接-httpspubdev-报错解决)
- [flutter 没有添加到path](#flutter-没有添加到path)
- [Unable to locate Android SDK.](#unable-to-locate-android-sdk)
  - [方法一：配置Android SDK环境变量](#方法一配置android-sdk环境变量)
  - [方法二：配置flutter](#方法二配置flutter)
- [gradle 配置镜像](#gradle-配置镜像)


# Flutter 连接 https://pub.dev 报错解决

> https://docs.flutter.cn/get-started/learn-flutter

- 方法1： 配置环境变量
```powershell
$env:PUB_HOSTED_URL="https://pub.flutter-io.cn"
$env:FLUTTER_STORAGE_BASE_URL="https://storage.flutter-io.cn"
```

- 方法2： 修改 http_host_validator.dart 文件
路径：`{Flutter安装路径}\flutter\packages\flutter_tools\lib\src\http_host_validator.dart`
修改常量： `kPubDev ` 

# flutter 没有添加到path

1. 找到flutter sdk路径: `{flutter安装路径\flutter}`
2. 打开环境变量
    - windows: 
        1. 路径：`系统->高级系统设置->环境变量`
        2. 找到path变量，添加flutter sdk路径 `{flutter安装路径}\flutter\bin`
    - macos/linux: 
        1. 路径 `~/.bashrc OR ~/.profile`
        2. 添加 `export PATH=$PATH:{flutter安装路径}\flutter\bin`
        3. 保存并运行  `source ~/.bashrc OR ~/.profile`
3. 验证安装
```shell
flutter doctor
```

# Unable to locate Android SDK.
## 方法一：配置Android SDK环境变量
1. 找到Android SDK路径: `{Android SDK安装路径}`
2. 确保路径下有文件夹 `cmdline-tools`
3. 添加环境变量 `ANDROID_HOME`
4. 重启电脑，并验证
```shell
flutter doctor --android-licenses
```

## 方法二：配置flutter
1. 运行命令
```shell
flutter config --android-sdk {Android SDK安装路径}
```
2. 验证
```shell
flutter doctor --android-licenses
```

# gradle 配置镜像
> 理论上下面三种都可以，但我这里直接都配置了。

插个题外话，**建议先配置 gradle 的缓存位置。** [参考](../包管理工具/gradle/gradle配置.md)


1. 编辑 `android/build.gradle` 文件，添加镜像源
```gradle
allprojects {
  repositories {
      // 阿里云镜像（按顺序优先使用）
      maven { url 'https://maven.aliyun.com/repository/gradle-plugin' }
      maven { url 'https://maven.aliyun.com/repository/google' }
      maven { url 'https://maven.aliyun.com/repository/jcenter' }
      maven { url 'https://maven.aliyun.com/repository/public' }
      
      // 官方仓库（备用）
      google()
      mavenCentral()
  }
}
```
2. 编辑 `android/gradle/wrapper/gradle-wrapper.properties` 中的`distributionUrl`
```properties
distributionUrl=https://mirrors.aliyun.com/gradle/distributions/v8.12.1/gradle-8.12.1-all.zip
# 可以上https://mirrors.aliyun.com/gradle/distributions选择想要的版本
```
3. 编辑 `android/gradle.properties` 文件，使用代理
```properties
# 使用阿里云的Maven仓库作为镜像
systemProp.http.proxyHost=mirrors.aliyun.com
systemProp.http.proxyPort=80
systemProp.https.proxyHost=mirrors.aliyun.com
systemProp.https.proxyPort=80
 
# 或者直接设置Gradle的仓库镜像
org.gradle.daemon=true
org.gradle.jvmargs=-Xmx1536M
android.useAndroidX=true
android.enableJetifier=true
systemProp.mavenCentralMirror=https\://maven.aliyun.com/repository/public/
```
4. 验证
```shell
cd ./android
./gradlew build --refresh-dependencies
```