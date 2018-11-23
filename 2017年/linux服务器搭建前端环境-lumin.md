# Linux下前端构建环境安装
## Node.js和NPM安装
以ROOT用户：
1. 下载Nodejs安装包（Binaries）：https://nodejs.org/en/download/，并解压；

2. 把解压后的Nodejs目录下的/bin目录添加到PATH环境变量；

3. node -v 和 npm -v 验证是否安装成功
```
$ node -v
$ npm -v
```



## 项目代码及包安装
1. 以root运行下面的命令：
```
$ npm install jshint@^2.9.4 eslint@^3.19.0 webpack@^2.3.2 gulp@^3.9.1 cross-env@^3.2.4 -g --registry=https://registry.npm.taobao.org
```
5. 以publish账号登录，并git clone 前端项目
```
$ su - publish
$ git clone git@xxxxxxx:root/xxxx-front.git
```

6. 以publish进入aicai-front，执行命令：sh bin/node-install.sh 安装前端项目包文件
```
$ cd aicai-front
$ sh bin/node-install.sh
```
7. aicai-front目录下执行前端编译
```
$ sh bin/cmd.sh build cps-static-app,cps-static-h5
```

## 静态服务nginx部署
1. 进入nginx系统默认安装配置中心
```
$ cd /usr/local/ngnix/conf
```
2. 将SSL证书文件xxxx.key,xxxx.pem拷贝到当前目前
```
$ rz
```

3. 修改conf目录下的配置文件nginx.conf
```
$ vi nginx.conf
```
添加和server节点并列的项：
```
include conf.d/*.conf;
```

4. 新建一个目录conf.d，将server-test.conf文件拷贝到conf.d目录
5. 确保每个项目的root路径和aicai-front的路径匹配，如果是通过发布平台发布，一般是在/mnt/dat1/aicai/下，当然也可以会不一样。
6. 测试并启动nginx
```
$ /usr/local/nginx/sbin/nginx -t  #测试配置文件
$ /usr/local/nginx/sbin/nginx     #启动
$ /usr/local/nginx/sbin/nginx -s reload #重启
```

