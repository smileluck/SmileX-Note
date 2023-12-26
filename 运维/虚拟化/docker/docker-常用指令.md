[toc]

---

# 【docker运行jar】

打包docker镜像

docker build -t my-app .

交互执行镜像（1800:8888，主机的端口：容器的端口）

 docker run -it --rm --name my-running-app -p 1800:8888 my-app

后台运行

 docker run -d --rm --name my-running-app -p 1800:8888 my-app

# 【容器管理】

所有容器信息

docker container ls

停止容器

docker container stop c97dcbc8664b（container Id）

启动容器

docker container start c97dcbc8664b（container Id）

重启容器

docker container rm c97dcbc8664b（container Id）

删除容器(必须容器已经停止)

docker container rm c97dcbc8664b（container Id）

查看运行容器

docker ps [-l 查询最后启动的容器]

查看容器端口映射

docker port c97dcbc8664b（containerid）

查看容器日志

docker logs -f c97dcbc8664b（container id）

查看容器的进程

docker top c97dcbc8664b（container Id)

查看 Docker 的底层信息

docker inspect c97dcbc8664b（container Id)

容器交互

docker exec -it c97dcbc8664b bash

# 【镜像管理】

所有镜像

docker images

获取镜像

docker pull mysql

查找镜像

docker search mysql 

删除镜像 

docker rmi [-f 强行删除] c97dcbc8664b（images id）

（删除不了时，查看一下dockers ps -a ，如果容器已经停止，但这里依然查出来，执行docker rm containerid 删除后，在执行docker rmi imageid）

# 【docker容器和主机互相拷贝文件】

语法

docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-

docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH

实例

将主机./RS-MapReduce目录拷贝到容器30026605dcfe的/home/cloudera目录下。

docker cp RS-MapReduce 30026605dcfe:/home/cloudera

将容器30026605dcfe的/home/cloudera/RS-MapReduce目录拷贝到主机的/tmp目录中。

docker cp  30026605dcfe:/home/cloudera/RS-MapReduce /tmp/