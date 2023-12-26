[toc]

---

# gitlab和服务同一台机器

> 无Docker服务

## .gitlab-ci.yml

```yaml
stages:
  - test_cloud_build
  - test_cloud_stop
  - test_cloud_start
  
develop_build_cloud:
  stage: test_cloud_build
  only:
  # 只有在tag 为develop开头生效
    - /^develop-[[:digit:]].*/
  tags:
  # CI 进程名称。gitlab需要自己开启和安装
    - ci-runner
  script:
    - JAVA_HOME=/usr/local/java/jdk1.8.0_261
    - CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
    - GRADLE_HOME=/usr/local/gradle-5.2.1
    - PATH=$PATH:$HOME/.local/bin:$HOME/bin:$GRADLE_HOME/bin
    - export JAVA_HOME
    - export GRADLE_HOME
    - export CLASSPATH
    - export PATH
    - gradle clean
    - module1=cloud-comsumer
    - package_name=cloud-test.jar
    - /usr/local/gradle-5.2.1/bin/gradle bootJar -p ${module1}
    - echo "$CI_COMMIT_REF_NAME"
    - echo "cp ${module1}/build/libs/*.jar /home/ubuntu/serve/cloud/${package_name}"
    - sudo cp ${module1}/build/libs/*.jar /home/ubuntu/serve/cloud/${package_name}
develop_stop_cloud:
  stage: test_cloud_stop
  only:
    - /^develop-[[:digit:]].*/
  tags:
    - ci-runner
  script:
    - sudo systemctl stop cloud-test
develop_start_cloud:
  stage: test_cloud_start
  only:
    - /^develop-[[:digit:]].*/
  tags:
    - ci-runner
  script:
    - sudo systemctl start cloud-test
```

## service

1. 进入system目录下，并编辑test.service文件

```shell
# 到system目录下
cd /etc/systemd/system

# 编辑test.service
vim test.service
```

2. 文件内容如下

```sh
[Unit]
Description=cloud-test-service server daemon
After=network-online.target
Wants=network-online.target

[Service]
User=root
Group=root
ExecStart=/bin/sh -c 'exec java -jar /home/ubuntu/serve/cloud/cloud-test.jar --spring.profiles.active=test --logging.file=/var/log/cloud-test.log'
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failur
RestartSec=60s

[Install]
```

3. 保存文件并注册服务。更多指令可查看systemctl.md文件。

```shell
# 重新加载所有被修改过的服务配置，否则配置不会生效
sudo systemctl daemon-reload

# 系统开机时自动启动指定unit，前提是配置文件中有相关配置
sudo systemctl enable test.service

# 启动服务
sudo systemctl start test.service

# 停止服务
sudo systemctl stop test.service

# 重启服务
sudo systemctl restart test.service

# 重新加载配置
sudo systemctl reload test.service
```

