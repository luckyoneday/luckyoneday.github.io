---
title: "HTTP、浏览器面试知识点汇总"
date: 2021-04-17T11:58:56+08:00
lastmod: 2021-04-17T11:58:56+08:00
draft: false
toc: true
keywords: ["interview"]
description: "HTTP、浏览器面试知识点汇总"
tags: ["interview"]
author: "youting"
---

## TCP/IP 协议

计算机之间的通信要遵循不同的协议，来实现互相之间的通信。

- 应用层：DNS 协议、HTTP、FTP 等。
- 传输层：TCP、UDP 协议，识别端口号。
  - UDP 是 IP 协议在传输层暴露的接口，简单不可靠，实现数据包形式的通信，不能保证数据包到达的顺序。不需要经过握手创建连接的过程。支持多播和广播。
  - TCP 实现流形式的传输，虚拟了文本流的通信。
- 网络层：需要满足：1.能从物理层上在两个网络之间接收和发送 0/1 序列；2.能同时理解两种网络的帧格式。传输遵循 IP 协议的 IP 数据包，数据包携带有出发地和目的地的 IP 地址，最终允许 IP 包从互联网的一台电脑传送到另一台。IP 协议，提供点到点的服务
- 连接层：信息以帧为单位传输，作用就是识别 0/1 序列所包含的帧。**以太网**和 **WIFI** 是现在最常用的连接层协议。
- 物理层：光纤、电缆或者电磁波等真实存在的物理媒介。对于数字应用来说发送和解析 0 和 1 就 ok。

### TCP 协议

- TCP 提供一种面向连接的、可靠的字节流服务
- 在一个 TCP 连接中，仅有两方进行彼此通信。广播和多播不能用于 TCP
- TCP 使用校验和，确认和重传机制来保证可靠传输
- TCP 给数据分节进行排序，并使用累积确认保证数据的顺序不变和非重复
- TCP 使用滑动窗口机制来实现流量控制，通过动态改变窗口的大小进行拥塞控制

### 三次握手四次挥手

{{% admonition info "三次握手" %}}

- 第一次握手(SYN=1, seq=x)，发送完毕后，客户端进入 SYN_SEND 状态。
- 第二次握手(SYN=1, ACK=1, seq=y, ACKnum=x+1)，发送完毕后，服务器端进入 SYN_RCVD 状态。
- 第三次握手(ACK=1，ACKnum=y+1)，发送完毕后，客户端进入 ESTABLISHED 状态，当服务器端接收到这个包时，也进入 ESTABLISHED 状态，TCP 握手结束。

{{% /admonition %}}

{{% admonition info "四次挥手" %}}

- 第一次挥手(FIN=1，seq=x)，发送完毕后，客户端进入 FIN_WAIT_1 状态。
- 第二次挥手(ACK=1，ACKnum=x+1)，服务端发送完毕后，表明自己接受到了客户端关闭连接的请求，进入 CLOSE_WAIT 状态，客户端接收到后进入 FIN_WAIT_2 状态，等待服务器端关闭连接。
- 第三次挥手(FIN=1，seq=y)，服务器端准备关闭连接时，向客户端发送结束连接请求，FIN 置为 1。发送完毕后，服务器端进入 LAST_ACK 状态，等待来自客户端的最后一个 ACK。
- 第四次挥手(ACK=1，ACKnum=y+1)，客户端接收到来自服务器端的关闭请求，发送一个确认包，并进入 TIME_WAIT 状态，等待可能出现的要求重传的 ACK 包。服务器端接收到这个确认包之后，关闭连接，进入 CLOSED 状态。客户端等待了某个固定时间（两个最大段生命周期，2MSL，2 Maximum Segment Lifetime）之后，没有收到服务器端的 ACK ，认为服务器端已经正常关闭连接，于是自己也关闭连接，进入 CLOSED 状态。

{{% /admonition %}}

### socket

socket 是应用层与 TCP/IP 协议族通信的中间软件抽象层。

### OSI 协议

- 应用层：提供高级 API，直接对应用户行为，HTTP、FTP
- 展示层：也被称为语法层，将应用层的数据转化为传输格式，保留语义
- 会话层：提供管理会话的方法（Open/Close/ReOpen/检查状态等）
- 传输层：提供主机到主机（host-to-host）的数据通信能力
- 网络层：提供数据在逻辑单元（例如 IP 地址）之间的传递能力
- 数据链路层：提供数据在设备和设备之间的传输能力
- 物理层：定义底层一个位（bit）的数据如何让变成物理信号

## HTTP 和 HTTPS

### HTTP

无连接无状态的超文本传输协议。

- HTTP 请求组成：请求行、消息报头、请求正文
- HTTP 响应组成：状态行、消息报头、响应正文

### 状态码

- `200 OK` 客户端请求成功
- `301 Moved Permanently` 请求永久重定向
- `302 Moved Temporarily` 请求临时重定向
- `304 Not Modified` 文件未修改，可以直接使用缓存的文件
- `400 Bad Request` 由于客户端请求有语法错误，不能被服务器所理解
- `401 Unauthorized` 请求未经授权。这个状态代码必须和 WWW-Authenticate 报头域一起使用
- `403 Forbidden` 服务器收到请求，但是拒绝提供服务。服务器通常会在响应正文中给出不提供服务的原因
- `404 Not Found` 请求的资源不存在，例如，输入了错误的 URL
- `500 Internal Server Error` 服务器发生不可预期的错误，导致无法完成客户端的请求
- `503 Service Unavailable` 服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常

### 缓存

缓存的优点： 1. 减少响应延迟； 2.减少网络带宽消耗

浏览器缓存主要有两类：强制缓存和协商缓存，一旦命中强制缓存不会与服务端交互，直接将数据返回，而协商缓存无论有没有命中都会与服务器交互。

#### 强制缓存

对于强制缓存而言，响应 header 中会有两个字段来表示资源是否过期：`Expires/Cache-Control`

- `Expires` 指的是资源的过期时间，即下一次请求时，请求时间小于服务端返回的到期时间，直接使用缓存数据。但是由于服务端时间会和客户端时间有偏差，这个字段被 `Cache-Control` 字段替代；
- `Cache-Control`：
  - `max-age=xxx` 缓存的内容将在 xxx 秒后失效，若没有过期，则可以直接返回并使用
  - `no-cache` 需要使用对比缓存来验证缓存数据
  - `no-store` 所有内容都不会缓存，强制缓存，协商缓存都不会

### 协商缓存

浏览器第一次请求数据时，服务器会将缓存标识与数据一起返回给客户端，再次请求数据时，客户端将备份的缓存标识发送给服务器，服务器根据缓存标识进行判断，判断成功后，返回 304 状态码，通知客户端比较成功，可以使用缓存数据。

{{% admonition info "Etag / If-None-Match" %}}

- 服务器响应请求时，会带 `Etag` 字段，告诉浏览器当前资源在服务器的唯一标识
- 客户端再次发送请求时，携带 `If-None-Match` 字段，由服务端进行资源比对，若与之前的 Etag 不同，说明资源有新的修改，则返回状态码 200 并返回新的资源
- 若相同则说明资源没有修改，则返回状态码 304

{{% admonition info "提示" %}}

优先级高于 `Last-Modified / If-Modified-Since`

{{% /admonition %}}

{{% /admonition %}}

{{% admonition info "Last-Modified / If-Modified-Since" %}}

- 服务器响应请求时，会带 `Last-Modified` 字段，表示资源的最后修改时间
- 客户端再次发送请求时，携带 `If-Modified-Since` 字段，由服务端进行资源比对，若最后修改时间大于 `If-Modified-Since` 则说明资源有被修改过，则返回状态码 200 并返回新的资源
- 若最后修改时间等于或小于 `If-Modified-Since` 则说明资源没有修改，则返回状态码 304

{{% /admonition %}}

### 持久链接

客户端浏览器在请求头中增加 `Connection: Keep-Alive`，当服务器收到附带有 `Connection: Keep-Alive` 的请求时，它也会在响应头中添加一个同样的字段来使用 Keep-Alive 。这样一来，客户端和服务器之间的 HTTP 连接就会被保持，不会断开。

- 这个连接不可能一直保持，例如 `Keep-Alive: timeout=5, max=100`，表示这个 TCP 通道可以保持 5 秒，max=100，表示这个长连接最多接收 100 次请求就断开。
- 如何判断本次传输结束？
  - 判断传输数据是否达到了 `Content-Length` 指示的大小；
  - 动态生成的文件没有 `Content-Length`，它是分块传输（chunked），这时候就要根据 chunked 编码来判断，chunked 编码的数据在最后有一个空 chunked 块，表明本次传输数据结束。

### 管道化

管道化允许客户端在已发送的请求收到服务端的响应之前发送下一个请求，借此来减少等待时间提高吞吐。

但是管道化要求服务端按照请求发送的顺序返回响应（FIFO），原因很简单，HTTP 请求和响应并没有序号标识，无法将乱序的响应与请求关联起来，如果一个响应返回延迟了，那么其后续的响应都会被延迟，直到队头的响应送达。即会造成队头阻塞。

### 会话跟踪

对同一个用户对服务器的连续的请求和接受响应的监视。

Cookie 和 Session

在服务器端会创建一个 session 对象，产生一个 sessionID 来标识这个 session 对象，然后将这个 sessionID 放入到 Cookie 中发送到客户端，下一次访问时，sessionID 会发送到服务器，在服务器端进行识别不同的用户。

### 安全

{{% admonition info "CSRF：跨站请求伪造" %}}

避免的话需要过滤请求来源：

- 关键操作只支持 POST 请求
- 一些关键步骤设置验证码
- 检测 Referer
- 设置 token，在某些请求设置 token，某些操作验证 token，token 有有效期

{{% /admonition %}}

{{% admonition info "XSS：跨站脚本攻击" %}}

- 需要过滤用户输入，对输入进行 escape。
- 开启 CSP `Content-Security-Policy`
  - 只允许加载本站资源 `Content-Security-Policy: default-src 'self'`
  - 只允许加载 HTTPS 协议图片 `Content-Security-Policy: img-src https://\*`
- `cookie` 设置 `HttpOnly`

{{% /admonition %}}

### TLS 加密

- 客户端发起请求（ClientHello）支持的协议版本、加密方法、压缩方法
- 服务端回应（ServerHello）确认使用的协议版本、加密方法，返回加密公钥和服务器证书
- 客户端回应，验证证书是否可信，如果可信，就会生成一段随机数并用公钥加密，随后发送给服务端
- 服务端使用自己的私钥将信息解密取出随机数，然后使用这串随机数生成自己的对称主密钥并返回客户端
- 客户端发送一个 Finished 消息给服务器端，使用对称密钥加密这次通讯的一个散列值
- 服务器端生成自己的 hash 值，然后解密客户端发送来的信息，检查这两个值是否对应。如果对应，就向客户端发送一个 Finished 消息，也使用协商好的对称密钥加密

接下来整个 TLS 会话都使用对称秘钥进行加密，传输应用层（HTTP）内容，即 HTTPS。

{{% admonition info "中间人攻击" %}}

攻击者与通讯的两端分别建立联系，使通讯的两端认为他们正在通过一个私密的连接与对方直接对话，但事实上整个会话都被攻击者完全控制。

SSL 剥离：阻止用户使用 https 访问网站。解决方案是强制浏览器使用 HTTPS 访问网站。

伪造证书攻击：需要证书颁发机构能够控制自己不滥发证书。

{{% /admonition %}}

### HTTP 和 HTTPS 的区别

- 默认端口号就不一样，HTTP 的默认端口号为 80，HTTPS 的默认端口号为 443
- HTTP 在传输过程中使用的是明文 传输，内容可能被窃取，而且无法验证通信方的身份，还有可能遭遇身份伪装，而 HTTPS 在应用层和传输层之间增加了 ssl 协议用来加密 内容，因此通过证书验证来验证身份，即使数据被窃取也无法解密，数据的传输更加安全。

### HTTP2.0 和 HTTP1.x 相比的新特性

- 二进制分帧层：HTTP1.x 以换行符作为纯文本的分隔符，而 HTTP2.0 将所有传输的信息分割为更小的消息和帧，并对它们采用二进制格式的编码。
- 多路复用：新的二进制分帧层突破了这些限制，实现了多向请求和响应：客户端和服务器可以把 HTTP 消息分解为互不依赖的帧，然后乱序发送，最后再在另一端把它们重新组合起来。可以带来很多好处：
  - 可以并行交错的发送请求，请求之间互不影响；
  - 可以并行交错的发送响应，响应之间互不干扰；
  - 只使用一个连接即可并行发送多个请求和响应；
  - 消除不必要的延迟，从而减少页面加载的时间；
  - 不必再为绕过 HTTP1.x 限制而多做很多工作。
- 压缩报头，减少开销。

## 输入一个 url 按下回车后发生了什么

[输入一个 url 按下回车后发生了什么](https://zhuanlan.zhihu.com/p/133906695)

## 同源 和 跨域

同源就是指 URL 中 protocol 协议、host 域名、port 端口这三个部分相同

同源策略限制：

- AJAX
- 无法获取 DOM 元素并操作
- 无法获取 localStorage、Cookie、IndexDB

某些请求不受同源策略限制：WebSocket、script、img、iframe、video、audio 标签的 src 属性等。

怎么跨域请求：

### 代理

前端代理可以使用 Webpack devServer proxy 功能：

```js
config.devServer = {
  hot: true,
  historyApiFallback: true,
  contentBase: "./dist",
  proxy: {
    "/api": {
      target: "http://localhost:2333",
      changeOrigin: true,
    },
  },
};
```

或者是 charles 转发。

### JSONP

动态创建一个 `script`，并将该标签的 src 属性指向跨域的接口，并将 callback 函数名作为请求的参数，只支持 get 请求。

```js
// 1. 创建回调函数callback
function myCallback(res) {
  alert(JSON.stringify(res, null, 2));
}
document.getElementById("btn-4").addEventListener("click", function () {
  // 2. 动态创建script标签，并设置src属性，注意参数cb=myCallback
  var script = document.createElement("script");
  script.src = "http://127.0.0.1:3000/info/jsonp?cb=myCallback";
  document.getElementsByTagName("head")[0].appendChild(script);
});
```

```js
// 接口请求
router.get("/jsonp", (req, res, next) => {
  var str = JSON.stringify(data);
  // 3. 创建script脚本内容，用`callback`函数包裹住数据
  // 形式：callback(data)
  var script = `${req.query.cb}(${str})`;
  res.send(script);
});
// 4. 前端收到响应数据会自动执行该脚本
```

### CORS

浏览器在请求头信息中添加 `Origin` 字段标识请求的来源。

服务端响应信息如下：

```
Access-Control-Allow-Origin: http://xxx.com // 表示接受的跨域请求来源，* 表示接受所有域的请求
Access-Control-Allow-Credentials: true // 值为 true，表示允许浏览器发送 Cookie 到服务器，客户端里需要设置withCredentials 属性为 true，表示带 Cookie
```

请求源不在服务器允许范围内，服务器会返回一个正常的（不带上述几个字段）的响应。浏览器发现响应头中没有 `Access-Control-Allow-Origin` 字段，则在 XMLHttpRequest 对象的 `onerror` 回调函数中捕捉错误（这种错误无法通过 HTTP 的状态码来识别，状态码有可能是 200）
