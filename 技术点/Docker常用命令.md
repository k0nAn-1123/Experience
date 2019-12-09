# Docker常用命令 #
## 容器生命周期管理 ##
### run ###
启动一个镜像。  
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]  
### start/stop/restart ###
docker start :启动一个或多个已经被停止的容器  
docker start [OPTIONS] CONTAINER [CONTAINER...]  
docker stop :停止一个运行中的容器  
docker stop [OPTIONS] CONTAINER [CONTAINER...]  
docker restart :重启容器  
docker restart [OPTIONS] CONTAINER [CONTAINER...]  
### kill ###
停止一个进程  
docker kill [OPTIONS] CONTAINER [CONTAINER...]
### rm ###
删除一个或多个容器  
docker rm [OPTIONS] CONTAINER [CONTAINER...]  
### pause/unpause ###
docker pause :暂停容器中所有的进程。  
docker pause [OPTIONS] CONTAINER [CONTAINER...]  
docker unpause :恢复容器中所有的进程。  
docker unpause [OPTIONS] CONTAINER [CONTAINER...]  
### create ###
创建一个新的容器但不启动  
docker create [OPTIONS] IMAGE [COMMAND] [ARG...]
### exec ###
在运行的容器中执行命令  
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]  
这个命令就是进入容器内部，用命令行环境操作容器，在这个命令中使用exit退出容器不会导致容器的生命周期停止。
## 容器操作 ##
### ps ###
列出所有容器  
docker ps [OPTIONS]
### inspect ###
获取容器的元数据  
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
### top ###
查看容器中运行的进程信息，支持 ps 命令参数  
docker top [OPTIONS] CONTAINER [ps OPTIONS]
### attach ###
连接到正在运行中的容器  
docker attach [OPTIONS] CONTAINER  
(这个命令和exec的效果一样，不同的是通过exec命令进入容器再使用exit命令退出后，不会导致容器停止生命周期，而attach命令会。)
### events ###
从服务获取实时事件  
docker events [OPTIONS]
### logs ###
获取容器的日志
docker logs [OPTIONS] CONTAINER
### wait ###
阻塞运行直到容器停止，然后打印出它的退出代码  
docker wait [OPTIONS] CONTAINER [CONTAINER...]
### export ###
将文件系统作为一个tar归档文件导出到STDOUT  
docker export [OPTIONS] CONTAINER
### port ###
列出指定的容器的端口映射，或者查找将PRIVATE_PORT NAT到面向公众的端口  
docker port [OPTIONS] CONTAINER [PRIVATE_PORT[/PROTO]]
## 容器rootfs命令 ##
### commit ###
从容器创建一个新的镜像。
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
### cp ###
用于容器与主机之间的数据拷贝。  
docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-  
docker cp [OPTIONS] SRC_PATH|- CONTAINER:DEST_PATH
### diff ###
检查容器里文件结构的更改。  
docker diff [OPTIONS] CONTAINER
## 镜像仓库 ##
### login/logout ###
docker login : 登陆到一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub  
docker login [OPTIONS] [SERVER]  
docker logout : 登出一个Docker镜像仓库，如果未指定镜像仓库地址，默认为官方仓库 Docker Hub  
docker logout [OPTIONS] [SERVER]
### pull ###
从镜像仓库中拉取或者更新指定镜像  
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
### push ###
将本地的镜像上传到镜像仓库,要先登陆到镜像仓库  
docker push [OPTIONS] NAME[:TAG]
### search ###
从Docker Hub查找镜像  
docker search [OPTIONS] TERM
## 本地镜像管理 ##
### images ###
列出本地镜像  
docker images [OPTIONS] [REPOSITORY[:TAG]]
### rmi ###
删除本地一个或多少镜像  
docker rmi [OPTIONS] IMAGE [IMAGE...]
### tag ###
标记本地镜像并将其归入某一仓库  
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
### build ###
命令用于使用 Dockerfile 创建镜像  
docker build [OPTIONS] PATH | URL | -
### history ###
查看指定镜像的创建历史  
docker history [OPTIONS] IMAGE
### save ###
将指定镜像保存成 tar 归档文件  
docker save [OPTIONS] IMAGE [IMAGE...]
### load ###
导入使用 docker save 命令导出的镜像  
docker load [OPTIONS]
### import ###
从归档文件中创建镜像  
docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
## info|version ##
### info ###
显示 Docker 系统信息(包括镜像和容器数)  
docker info [OPTIONS]
### version ###
显示 Docker 版本信息  
docker version [OPTIONS]