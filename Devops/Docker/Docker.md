# Docker

## 常用命令

### 镜像命令

```
docker images 查看本地镜像
docker rmi 删除镜像
docker pull 
docker push 
docker save 导出镜像
docker load 
```

### 容器命令

```
docker run
docker pause
docker unpause
docker stop
docker start

docker ps 查看所有运行的容器及状态
docker logs 查看容器允许日志
docker exec 进入容器执行命令
docker rm 删除指定容器
```

### 数据卷

**卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷。**

​	数据卷是一个可供一个或多个容器使用的特殊目录，它绕过UFS,可以提供很多有用的特性： 

- 数据卷可以在容器之间共享和重用 
- 对数据卷的修改会立马生效 
- 对数据卷的更新，不会影响镜像
- 数据卷默认会一直存在，即使容器被删除

1. 挂载数据卷 : ```-v volume 名称 :容器内目录```
2. 挂载宿主机文件 : ```-v 宿主机文件/目录 ：容器内文件/目录```

### 自定义镜像

#### 镜像的结构

分层结构，每一层成为一个layer

- BaseImage 层：包含基本的系统函数库、环境变量、系统文件
- EntryPoint： 入口，是镜像中应用启动的命令
- 其他： 在BaseImage 基础上添加依赖、安装程序、完成整个应用的安装和配置。

#### 自定义镜像

```docker
docker build 用于使用 Dockerfile 创建镜像。
```



Dockerfile 是一个文本文件，其中包含一个个指令，指令说明来用于构建镜像的操作，每个指令都会形成一层layer。

| 指令       | 说明                                       | 示例                        |
| ---------- | ------------------------------------------ | --------------------------- |
| FROM       | 指定基础镜像                               | FROM centos:7               |
| ENV        | 设置环境变量，可在后面指令使用             | ENV key value               |
| COPY       | 拷贝本地文件到镜像的指定目录               | COPY  ./mysql-5.7.rpm /tmp  |
| RUN        | 执行Linux的shell命令，一般是安装过程的命令 | RUN yum install gcc         |
| EXPOSE     | 指定容器运行时监听的端口                   | EXPOSE 8080                 |
| ENTRYPOINT | 镜像中的应用的启动命令，容器运行时调用     | ENTRYPOINT java -jar xx.jar |



### DockerCompose

Compose文件是一个文本文件，
