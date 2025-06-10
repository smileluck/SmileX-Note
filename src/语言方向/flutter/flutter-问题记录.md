
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