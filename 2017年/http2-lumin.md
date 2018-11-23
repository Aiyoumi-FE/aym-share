# 认识 HTTP/2

HTTP/2 是 HTTP 协议自HTTP 1.1 发布后的升级，主要基于Google 的 SPDY 协议。 HTTP/2标准于2015年5月以[RFC 7540](https://httpwg.github.io/specs/rfc7540.html "RFC 7540")正式发表。

HTTP/2当前已经被大多数主流浏览器支持，且很多网站已经通过该协议实现。 例如taobao.com使用的是HTTP/2协议。
![](https://i.imgur.com/KeXFMqy.png)

## HTTP/2的基本概念
http2和现有的URI结构相同，在使用上没什么区别

### 二进制
- http2是一个二进制协议，不同于HTTP1.1的基于文本的协议，二进制协议解析起来更高效
- 帧：HTTP/2数据通信的最小单位。帧用来承载特定类型的数据，如 HTTP 首部、负荷；或者用来实现特定功能，例如打开、关闭流。每个帧都包含帧首部，其中会标识出当前帧所属的流；
- 消息（Message）：指 HTTP/2 中逻辑上的 HTTP 消息。例如请求和响应等，消息由一个或多个帧组成。
- 流（Stream）：存在于连接中的一个虚拟通道。流可以承载双向消息，每个流都有一个唯一的整数 ID；

### 多路复用传输（Multiplexing）
多路复用，允许浏览器在单个TCP连接中包含多个请求，从而使浏览器能够并行地请求所有的资源；HTTP1.X中，如果想并发多个请求，必须使用多个TCP连接，浏览器为了控制资源访问，对单个域名有6-8个的限制。

### 服务器推送（Server push）
服务器可以在浏览器知道需要该资源前，推送给浏览器（如：CSS、JS、Image），从而通过减少请求数量来加速页面加载时间；在HTTP1.x中，需要先加载HTML，然后解析HTML发现页面中的JS/CSS/Images资源，再发送请求去获取相应的资源，在HTTP/2中，在发送HTML到客户端时同时把这些JS/CSS/Images推送到客户端，减少了网络请求。主动推送遵循同源策略，即不能推送第三方资源（CDN）。

### 头部压缩（Header compression）
HTTP/2对消息头采用HPACK（专为http2头部设计的压缩格式）进行压缩传输，能够节省消息头占用的网络的流量。而HTTP/1.x每次请求的头部总是重复一样的内容，都会携带大量冗余头信息，浪费了很多带宽资源。


## 支持情况
浏览器支持情况：
![](https://i.imgur.com/Vm6IOE8.png)

HTTP/2的实现：
1. [http2-spec](https://github.com/http2/http2-spec/wiki/Implementations)
2. [HTTP/2 Now Fully Supported in NGINX Plus](https://www.nginx.com/blog/http2-r7/)
3. [Node.js 8.4.0 内置了http2模块](https://nodejs.org/api/http2.html）

## DEMO
![](https://i.imgur.com/ia2UYdm.png)
> https://http2.akamai.com/demo

## 对开发的影响
大部分适用于 HTTP/1.1 的优化技巧在 HTTP/2 中变成多余的，其中一些甚至反而会影响 HTTP/2 上的网站性能，例如：

1. 资源文件合并就没必要了；
2. 使用精灵图（image sprites）、CSS和JS打包，因为只要其中一小部分有改动就会影响客户端的缓存的作用；在 HTTP/2 协议上更好的方式是使用多个的小文件，而不是一个大文件。
3. 另一个适用于 HTTP/1.1 不适用于 HTTP/2 的是，域名分片（为了绕过TCP并行请求数量限制）。之所以建议不在 HTTP/2 使用域名分片，还因为每个域名会带来额外的查询负载。如果真的有需要，那么更好的方式是解析多个域名到同一个IP，而且保证你使用的是通配符证书或整合了多域名的证书，从而减少域名查询的时间。

## 小结
HTTP/2 拥有很多很好的特性，头部压缩，减少重复的头部数据；多路复用能并行的发送多个服务器资源；服务器推送可以在浏览器知道需要该资源前，推送给浏览器，减少网络请求。

##　参考资源
1. [Easy HTTP/2 Server with Node.js and Express.js](https://webapplog.com/http2-node/)
1. [HTTP/2 Now Fully Supported in NGINX Plus](https://www.nginx.com/blog/http2-r7/)
1. [HTTP/2 Push: The details](https://calendar.perfplanet.com/2016/http2-push-the-details/)
2. [HTTP/2 DEMO](https://http2.akamai.com/demo)
3. [HTTP/2 explained](https://bagder.gitbooks.io/http2-explained/zh/)
4. [Real–world HTTP/2: 400gb of images per day](https://99designs.com.au/tech-blog/blog/2016/07/14/real-world-http-2-400gb-of-images-per-day/)