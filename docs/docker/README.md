## docker

#### docker常见命令

**`查看`**
	
- `docker images`： 列出所有镜像(images)

- `docker ps`：	 列出正在运行的容器(containers)

- `docker ps -a`:  列出所有的容器

- `docker pull <image_name>`：  下载镜像

- `docker top <id、container_name>`：查看容器内部运行程序

- `docker logs <id、container_name>`：查看容器日志

**`容器`**
	
- `docker exec -it <id、container_name> /bin/bash`：进入容器

- `docker exec -it /bin/sh`：进入alpine类容器的sh

- `docker stop <id、container_name>`: 停止一个正在运行的容器

- `docker start <id、container_name>`：启动一个已经停止的容器

- `docker restart <id、container_name>`: 重启容器

- `docker rm <id、container_name>`：删除容器

- `docker run -i -t -p :80 LAMP /bin/bash`:	运行容器并做http端口转发

- `docker rm docker ps -a -q`：删除所有已经停止的容器

- `docker kill $(docker ps -a -q)`：杀死所有正在运行的容器，$()功能同``

**`镜像`**

- `docker history <image_name>`：显示一个镜像的历史	
	
- `docker build -t <image_name:tag> .`：构建1个镜像, -t(镜像的名字及标签) <id、container_name>(镜像名) .(构建的目录)

- `docker run -i -t <image_name>`：	-t -i以交互伪终端模式运行，可以查看输出信息

- `docker run -d -p 80:80 <image_name>`：镜像端口 -d后台模式运行镜像

- `docker rmi <image_id>`：删除镜像

- `docker rmi $(docker images -q)`：删除所有镜像

- `docker rmi $(sudo docker images --filter "dangling=true" -q --no-trunc)`：删除无用镜像


#### docker-compose怎么挂载磁盘

```dockerfile
 volumes:
   - <宿主机>:<容器>
```

#### 同⼀个宿主机中多个Docker容器之间如何通信？多个宿主机中Docker容器之间如何通信？

- 1、这⾥同主机不同容器之间通信主要使⽤Docker桥接（Bridge）模式。
- 2、不同主机的容器之间的通信可以借助于 `pipework` 这个⼯具。