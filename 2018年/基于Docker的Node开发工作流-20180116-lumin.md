## 基于Docker的Node开发工作流

### 准备
- Node.js
- Docker（Mac or Windows用户安装Docker Machine）
- Docker Compose

> windows: https://docs.docker.com/toolbox/toolbox_install_windows/


### 本文的目标
1. 新建一个简单的Node.js应用，并使用docker运行此应用
2. 使用docker连接mongodb数据库
3. 使用docker compose编排服务，简化运行部署流程
4. 了解Docker基本应用和服务编排

### 用Express新建一个简单的Node.js应用
```
> mkdir docker-node & cd docker-node
> npm init -y
> npm i express nodemon -S
```

新建app.js

```js
var express = require('express');
var app = express();
app.get('/', function(req, res){
  res.send("Hello World");
});
app.listen(3000, function(){
  console.log('app listening on port 3000!');
});
```

在package.json中添加scripts节点
```
"scripts": {
    "start": "nodemon app.js"
  },
```
启动应用测试
```
> npm start
app listening on port 3000!
```

到此一个基本的应用完成了，现在我们把这个应用改造能够运行在容器中，我们需要一个Dockerfile来配置应用能够在容器中正常运行的环境。

创建一个Dockerfile文件
```
FROM node:8.9.1
MAINTAINER Lumin
RUN mkdir /app
WORKDIR /app
COPY package.json /app
RUN npm install
COPY . /app
EXPOSE 3000
CMD ["npm", "start"]
```

- FROM: 指定基础镜像，：后面是指定的版本，如果没有指定，会使用最新基础镜像版本
- MAINTAINER： 指定作者
- RUN： 在当前镜像基础上执行指定命令
- WORKDIR： 设置工作目录
- COPY： 复制文件到容器
- EXPOSE： 告诉 docker服务端容器对外映射的本地端口
- CMD：docker创建、启动容器时执行的命令

> 参考：http://www.docker.org.cn/dockerppt/114.html

接下来我们构建这个镜像然后运行
```
> docker build -t my-node-demo:0.1 .
# 查看镜像
> docker images
> docker run -it my-node-demo:0.1
app listening on port 3000!
```

Docker-Compose简化你的docker流程，通过一个配置文件来管理多个Docker容器，非常适合组合使用多个容器进行开发的场景。

新建一个docker-compose.yaml来定义编排你的应用服务
```
version: "2"
services:
  web:
    build: .
    ports:
      - "3000:3000"
    volumes:
      - ./:/app
```

```
> docker-compose up
```

执行docker-compose up来启动你的应用，它会根据docker-compose.yaml的设置来pull/run这个容器，然后再启动。


添加mongo数据库
```
npm i mongoose -S
```
app.js中添加
```
var mongoose = require('mongoose');
// DB setup
mongoose.connect('mongodb://mongo:27017');
```

在docker-compose中添加mongodb到服务节点下，并且通过links把两个服务连接起来，'mongo'服务名同时也会添加到容器的/etc/hosts中，这样我们代码中可以直接使用mongodb://mongo:27017

```
version: "2"
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
> 如果是windows用户，挂载文件时报文件找不到，请看解决方案https://github.com/docker/compose/issues/2548


执行docker-compose up会执行web和mongo两个服务。
- 定义了两个服务ｗｅｂ和ｍｏｎｇｏ
- image: 指定基础镜像
- build: 该参数指定Dockerfile文件的路径
- ports:　暴露端口，指定两者的端口（主机：容器），或者只是容器的端口（主机会被随机分配一个端口）
- links：　连接到其他服务中的容器，可以指定服务名称和这个链接的别名，或者只指定服务名称，此时，在容器内部，会在/etc/hosts文件中用别名创建一个条目
- external_links：连接到在这个docker-compose.yml文件或者Compose外部启动的容器，特别是对于提供共享和公共服务的容器
- volumes：挂载路径，可以选择性的指定一个主机上的路径（主机：容器），可以在不重新构建镜像的情况下修改代码。


### 结语
使用docker可以很方便的管理维护团队共享的镜像，简化操作流程，在团队开发中，任何人都可以直接拉取代码并运行docker-compose up来跑应用，并且运用的配置都是相同的。

关于Docker的功能远比这强大多，大家可以继续去官方学习，动手实践。

> docker文档：https://docs.docker.com

