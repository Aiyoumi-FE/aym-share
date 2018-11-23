title: Docker入门
speaker: LuMin
transition: slide2
files: /js/demo.js,/css/demo.css,/js/zoom.js
 
[slide]

# Docker入门
## 演讲者：LuMin

[slide]
## 内容
- 什么是Docker
- Docker部件
- Docker基础用法
- Dockerfile
- Docker有什么好处


[slide]
## 什么是Docker
 
- Docker是一个开源的引擎，是一个容器平台
- 基于进程容器的轻量级操作系统虚拟化解决方案
- 可以轻松的为任何应用创建一个轻量级的、可移植的、自给自足的容器

[slide]
## Docker VS VM
![](/images/docker/docker-vm.jpg)

[slide]
 ## Docker 组件与元素
- Docker Client : Docker提供给用户的客户端
- Docker Daemon： Docker服务的守护进程
- Docker images： Docker镜像 
- Docker Registry： Docker仓库 
- Docker containers： Docker容器

[slide]
 ## Docker 部件关系图
![](/images/docker/internal.png)


[slide]
 ## Docker 镜像
命名方式为：author/name:tag， 如 danielfu/ubuntu:14.04

- 镜像都是只读的
- 官方镜像一般不带作者名
- tag省略时，下载tag为latest的版本
- docker优先从本地查找镜像，若不存在，在从远程Registry下载镜像，如果也没有则提示不存在

[slide]
 ## Docker 仓库
Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库

- 公有:  Docker Hub
- 私有： 自建Docker仓库

[slide]
## Docker 容器
- 一个Docker容器包含了所有的某个应用运行所需要的环境
- 每一个 Docker 容器都是从 Docker 镜像创建的。
- Docker 容器可以运行、开始、停止、移动和删除。
- 每一个 Docker 容器都是独立和安全的应用平台
- Docker 容器是 Docker 的运行部分

[slide]
## Docker 安装
具体安装参考https://docs.docker.com/engine/installation/


> DOCKER TOOLBOX(Mac/Windows) https://docs.docker.com/toolbox/

```
> docker version
```

[slide]
## Docker 常见命令

[slide]
## 容器相关
- docker create # 创建一个容器但是不启动它
- docker run # 创建并启动一个容器
- docker stop # 停止容器运行，发送信号SIGTERM
- docker start # 启动一个停止状态的容器
- docker restart # 重启一个容器
- docker rm # 删除一个容器
- docker kill # 发送信号给容器，默认SIGKILL
- docker attach # 连接(进入)到一个正在运行的容器
- docker wait # 阻塞到一个容器，直到容器停止运行

[slide]
## 获取容器相关信息
- docker ps # 显示状态为运行（Up）的容器
- docker ps -a # 显示所有容器,包括运行中（Up）的和退出的(Exited)
- docker inspect # 深入容器内部获取容器所有信息
- docker logs # 查看容器的日志(stdout/stderr)
- docker events # 得到docker服务器的实时的事件
- docker port # 显示容器的端口映射
- docker top # 显示容器的进程信息
- docker diff # 显示容器文件系统的前后变化
- docker exec # 在容器里执行一个命令，可以执行bash进入交互式


[slide]
## 镜像操作
- docker images # 显示本地所有的镜像列表
- docker import # 从一个tar包创建一个镜像，往往和export结合使用
- docker build # 使用Dockerfile创建镜像（推荐）
- docker commit # 从容器创建镜像
- docker rmi # 删除一个镜像
- docker load # 从一个tar包创建一个镜像，和save配合使用
- docker save # 将一个镜像保存为一个tar包，带layers和tag信息
- docker history # 显示生成一个镜像的历史命令
- docker tag # 为镜像起一个别名

[slide]
## 镜像仓库(registry)操作
- docker login # 登录到一个registry
- docker search # 从registry仓库搜索镜像
- docker pull # 从仓库下载镜像到本地
- docker push # 将一个镜像push到registry仓库中


[slide]
## Dockerfile
Dockerfile包含创建镜像所需要的全部指令

[slide]
## Dockerfile
```
#从一个基础镜像构建新的镜像
FROM node:8.9.1
#维护者信息
MAINTAINER Min "xxxx@xxxxx.com"

#指定RUN、CMD与ENTRYPOINT命令的工作目录
WORKDIR /app

#将外部文件拷贝到镜像里
COPY ./package.json /app/

#非交互式运行shell命令
RUN npm install

#将外部文件拷贝到镜像里
ADD . /app/

#指定容器在运行时监听的端口
EXPOSE 3000

#Dockerfile只允许使用一次CMD指令
CMD node bin/www
```

[slide]
## dockerfile 最佳实践
- 使用.dockerignore文件
- 避免安装不必要的软件包
- 每个容器都跑一个进程
- 最小化层
- 创建缓存
 
[slide]
## 使用Docker Compose管理多个容器


[slide]
## 配置文件
用一个compose.yaml来定义你的应用服务，他们可以把不同的服务生成不同的容器中组成你的应用
```
# create a docker-compose.yml file
version: "3"
services:
  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - ./:/app
    links:
      - mongo
  mongo:
    image: mongo
    ports:
      - "27017:27017"
```
执行docker-compose up来启动你的应用

[slide]
## Yaml文件参考 
- image:镜像的ID
- build:直接从pwd的Dockerfile来build，而非通过image选项来pull
- links：连接到那些容器。每个占一行，格式为SERVICE[:ALIAS],例如 – db[:database]
- external_links：连接到该compose.yaml文件之外的容器中，比如是提供共享或者通用服务的容器服务。格式同links
- command：替换默认的command命令
- ports: 导出端口
- expose：导出端口，但不映射到宿主机的端口上。
- volumes：加载路径作为卷
- volumes_from：加载其他容器或者服务的所有卷
- env_file：从一个文件中导入环境变量
- net：容器的网络模式，可以为”bridge”, “none”, “container:[name or id]”, “host”中的一个
- dns：可以设置一个或多个自定义的DNS地址。


[slide]
## Docker 有什么好处
> Docker可以解决虚拟机能够解决的问题，同时也能够解决虚拟机由于资源要求过高而无法解决的问题

- 隔离性
- 环境标准化和版本控制
- 标准化应用发布，可以跨平台和主机使用
- 节约时间，快速部署和启动
- 持续部署与测试

[slide]
## 参考
- https://docs.docker.com/compose/
- http://www.docker.org.cn/dockerppt/114.html
- https://training.docker.com/docker-operations
- http://www.devopsschool.com/slides/docker/docker-web-development/index.html#/
