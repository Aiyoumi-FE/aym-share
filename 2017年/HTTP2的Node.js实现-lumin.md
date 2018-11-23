# 基于 Node.js 搭建 HTTP/2服务
上文已经提到了HTTP/2主要利用多路复用传输，头部压缩，服务端推送，可以减少网络延迟对性能带来的影响，优化首次访问速度，提高传输效率。

本文介绍基于Node.js搭建一个简单的HTTP/2应用，以及实现HTTP/2的Server Push特性，来直观的感受一下HTTP/2的特性。

### 准备SSL证书
1. 生成服务器私钥key
```
# 生成密码文件，省去输入密码
openssl genrsa -des3 -passout pass:123456 -out key/server.pass.key 2048 
openssl rsa -passin pass:123456 -in key/server.pass.key -out key/server.key
```

2. 生成证书请求文件CSR
```
openssl req -new -key key/server.key -out key/server.csr 
```

3. 生成证书
```
openssl x509 -req -days 3650 -in key/server.csr -signkey key/server.key -out key/server.crt
```

### Node.js代码实现
1. 新建项目
初始化项目并安装spdy和express：
```
npm init -y
npm i express spdy -D
```

2. 创建入口文件index.js，实例化express和引入spdy
```
const port = 3000
const spdy = require('spdy')
const express = require('express')
const path = require('path')
const fs = require('fs')
const app = express()
```

3. 读取证书
```
const options = {
    key: fs.readFileSync(__dirname + '/key/server.key'),
    cert: fs.readFileSync(__dirname + '/key/server.crt')
}
```

4. 把证书传入spdy并创建服务
```
app.get('/', (req, res) => {
    res.end('hello from http/2')
})
spdy
    .createServer(options, app)
    .listen(port, (error) => {
        if (error) {
            console.error(error)
            return process.exit(1)
        } else {
            console.log('Listening on port: ' + port + '.')
        }
    })
```

5. 启动服务
```
$ node index.js
```

6. 浏览器打开页面测试 https://127.0.0.1:3000， 因为我们创建的证书并没有经过CA机构认证，所以会提示页面不安全，点击继续访问即可。 查看网络请求协议可以看到网站使用了HTTP/2（h2）。
![](https://i.imgur.com/1koN9dN.png)

到此一个简易的基于Node.js的HTTP/2实现已经完成。 接下来我们看看HTTP/2协议中的Server Push功能。


### 服务端推送Server Push
为了改善延迟，HTTP/2引入了 Server Push ，这允许服务器在明确的请求之前将资源推送到浏览器。这就允许服务器充分利用空闲的网络来改善加载时间。

![](https://i.imgur.com/rQYxHVk.png)

添加服务端推送代码：
```
app.get('/', (req, res) => {
    var stream = res.push('/main.js', {
        status: 200, // optional
        method: 'GET', // optional
        request: {
            accept: '*/*'
        },
        response: {
            'content-type': 'application/javascript'
        }
    })
    stream.on('error', function() {})
    stream.end(fs.readFileSync('./public/main.js'))
    res.end(fs.readFileSync('./public/index.html'))
})
app.use('/', express.static(path.join(__dirname, 'public')))
```

网络请求可以看到main.js是由推送来的。在请求头部并没有太多的请求信息，因为和HTTP1.1不同，不需要客户端发起请求获取main.js，而是服务端在已经知道一个页面所需要的额外的资源，并且可以在响应初始请求时开始推送这些资源。

![](https://i.imgur.com/Vy7bdqR.png)

几个注意点：
1. 只能推送当前服务器上的资源，无法推送托管在CDN上的资源。
2. 除非客户端确定需要，否则不要推送，浏览器缓存的资源不要再推送。
3. 不要把客户端需要的所有资源的推送，根据实际情况按需推送。

### 参考资源
1. [HTTP/2 Push: The details](https://calendar.perfplanet.com/2016/http2-push-the-details/)
2. [HTTP/2 DEMO](https://http2.akamai.com/demo)
3. [HTTP/2 explained](https://bagder.gitbooks.io/http2-explained/zh/)

