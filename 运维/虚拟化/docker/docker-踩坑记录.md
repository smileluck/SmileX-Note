[toc]

---

# docker build

##  docker运行FROM java:8报错:manifest for java:8 not found : manifest unkown: manifest unknown解决方法     

dockerfile文件中的 `java:8` 替换成 `openjdk:8`

```dockerfile
from java:8
```

修改后

```dockerfile
from openjdk:8
```

## failed to solve with frontend dockerfile.v0: failed to create LLB definition:

windows中使用dockers build镜像，出现“failed to solve with frontend dockerfile.v0: failed to create LLB definition: failed to copy: httpReadSeeker: failed open: failed to do request:”。

解决方式：修改 `Docker Desktop` 的设置。（Docker桌面=>设置=>Docker Engine），将 `features.buildkit` 的值从 `true` 修改为 `false`

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "features": {
    "buildkit": true //this change
  }
}
```

修改后：

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "features": {
    "buildkit": false //this change
  }
}
```



#  The system cannot find the file specified. 

> 完成错误： error during connect: This error may indicate that the docker daemon is  not running.http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.24/version": open  //./pipe/docker_engine: The system cannot find the file specified.  

在 `win10` 环境下，执行 `docker ps`  会抛出这样的异常。根据意思是提权。所以我 换了 `CMD 管理员`，依然出现这个异常，网上有人说执行这个指令即可。

```shell
cd "C:\Program Files\Docker\Docker"
.\DockerCli.exe -SwitchDaemox
```

但依然不起作用，后来发现是我的 `Hyper-V` 关掉了。操作步骤

1. 打开 `控制面板`
2. 打开 `程序`
3. 选择`启用或关闭 Window 功能`
4. 开启 `Hyper-V` ，重启电脑即可。

可能还跟缓存日志有关。可以把日志清掉。



## Docker failed to initialize Docker Desktop is shutting down

原因的话应该是由于长时间未登陆导致log信息过期了。所以要修改/删除一下原来的信息。

不用重装docker。**将 C:\Users\YourUser\AppData\Roaming 目录下Docker目录重命名**。比如改为Docker_backup（这样做其实相当于删除了原信息但还把它里面的信息拷贝到备份里）。 