# http 和 https 的那些事

## 一、HTTP 和 HTTPS 的区别
http => 超文本传输协议，是互联网上应用最为广泛的一种网络协议，用于从www服务器传输超文本到本地浏览器的传输协议，使浏览器更加高效。
https =>  以安全为目标的HTTP通道，在http下加入SSL层，建立一个信息安全通道来确保数据传输。
区别：
|区别| http | https|
|-|-|-|
|-|信息明文传输|具有安全性的ssl加密传输|
|-|无状态连接|身份认证|
|-|不同的链接方式，端口默认80| 443端口|
|-||需要ca证书，费用较高|

## 二、https协议的工作原理
1）客户端使用http的url访问web服务器，要求与web服务器建立SSL连接；
2）Web服务器接收到请求之后，会将网站的证书信息(证书中包含公钥)返回给客户端；
3）客户端对照一下证书信息后，与web服务器协商SSL连接的安全等级；
4）客户端的浏览器根据双方的安全等级，建立会话密钥。利用公钥对信息加密，传送给web服务器；
5）web服务器通过私钥解密出会话信息，得到了客户端传过来的随机值(会话密钥)，然后把内容通过该值进行对称加密,从而与客户端通信。

## 三、HTTP1.0、HTTP1.1 和 HTTP2.0 的区别
HTTP发展历史：
|版本|时间|主要内容|备注|
|-|-|-|-|
|HTTP/0.9|1991|不涉及数据包传输，只能Get请求|没有正式作为标准|
|HTTP/1.0|1996|传输内容格式不受限制，增加Delete、Patch、HEAD、Options、Put|正式作为标准|
HTTP/1.1|1997|持久连接（长连接）、节约带宽、HOST域、管道机制、分块传输编码|2015年之前使用广泛|
|HTTP/2.0|2015|多路复用、服务器推送、头信息压缩、二进制协议等|

### 多路复用
> 通过单一的http/2连接请求发送多重的请求-响应信息，多个请求srteam 共享一个TCP连接，实现多路并行而不是依赖于建立多个TCP连接。

![多路复用](https://github.com/SweetyPeng/Pilgrim/blob/master/assets/什么是多路复用.png?raw=true)

在 HTTP 1.1 中，发起一个请求是这样的：
浏览器请求 url -> 解析域名 -> 建立 HTTP 连接 -> 服务器处理文件 -> 返回数据 -> 浏览器解析、渲染文件 

这个流程最大的问题是，每次请求都需要建立一次 HTTP 连接，也就是我们常说的3次握手4次挥手，这个过程在一次请求过程中占用了相当长的时间，而且逻辑上是非必需的，因为不间断的请求数据，第一次建立连接是正常的，以后就占用这个通道，下载其他文件，这样效率多高啊！

为了解决这个问题， HTTP 1.1 中提供了 Keep-Alive，允许我们建立一次 HTTP 连接，来返回多次请求数据。但是这里有两个问题：
1) HTTP 1.1 基于串行文件传输数据，因此这些请求必须是有序的，所以实际上我们只是节省了建立连接的时间，而获取数据的时间并没有减少
2) 最大并发数问题，假设我们在 Apache 中设置了最大并发数 300，而因为浏览器本身的限制，最大请求数为 6，那么服务器能承载的最高并发数是 50

而 HTTP/2 引入**二进制数据帧和流**的概念，其中帧对数据进行顺序标识，这样浏览器收到数据之后，就可以按照序列对数据进行合并，而不会出现合并后数据错乱的情况。同样是因为有了序列，服务器就可以并行的传输数据。HTTP/2 对同一域名下所有请求都是基于流，也就是说同一域名不管访问多少文件，也只建立一路连接。同样Apache的最大连接数为300，因为有了这个新特性，最大的并发就可以提升到300，比原来提升了6倍。

### 关于服务器推送 和 头信息压缩
服务器Push => 
在 http2.0 中，服务端可以在客户某个请求后，主动推送其他资源。比如某些资源客户端一定是会请求的，这时候可以采用服务端的push技术，提前给客户端推送必要的资源，这样减少延迟时间。

Header压缩 =>
http1.x 中，以文本的形式传输 header，在 header 携带 cookie 情况下，可能每次都需要重复传输几百到几千的字节;
在 http2.0 中，使用了 HPACK 压缩格式对传输的 header 进行编码，减少了 header 的大小。在网络中传输会更快。

## Tcp的三次握手和四次挥手
### 三次握手
=> 为了防止已失效的连接请求报文又重新传递

![tcp报文首部格式](https://github.com/SweetyPeng/Pilgrim/blob/master/assets/tcp报文首部格式.png?raw=true)

![tcp三次握手](https://github.com/SweetyPeng/Pilgrim/blob/master/assets/tcp三次握手.jpeg?raw=true)

### 四次挥手
![tcp四次挥手](https://github.com/SweetyPeng/Pilgrim/blob/master/assets/tcp四次挥手.jpeg?raw=true)

## 从输入URL到页面加载完成的过程
![从输入URL到页面加载完成的过程](https://github.com/SweetyPeng/Pilgrim/blob/master/assets/一次完整的http请求.png?raw=true)

### 域名解析
=> 浏览器解析域名得到对应的ip地址
分为以下：
 - 浏览器搜索自身DNS缓存，存在并且未过期，则结束，否则继续下一步；
- 搜索操作系统自身（本地）的DNS缓存，存在并且未过期，则结束，否则继续下一步；
- 读取Host文件，是否存在，存在，则结束，否则继续下一步；
- 浏览器发起DNS系统的调用，向本地配置的首选DNS服务器发起域名解析请求；
通过UDP协议向DNS的53端口发起递归请求，DNS协议工作在应用层，UDP协议工作在传输层

### 页面渲染的完整流程是怎样的
![页面渲染的完整流程](https://github.com/SweetyPeng/Pilgrim/blob/master/assets/页面渲染的完整流程.jpeg?raw=true)

浏览器内核拿到内容后，渲染步骤大致可以分为以下几步：
1）解析HTML，构建DOM树；
2）解析CSS，生成CSS规则树；
3）合并DOM树和CSS规则树，生成render树；
4）布局render树（layout/reflow），负责各元素尺寸、位置的计算；
5）绘制render树（paint），绘制页面像素信息；
6）浏览器会将各层的信息发送给GUI，GUI会将各层合成（composite），显示在屏幕上。

## HTTP请求报文和响应报文格式
### HTTP请求报文
![http请求报文格式](https://github.com/SweetyPeng/Pilgrim/blob/master/assets/http请求报文格式.png?raw=true)

常见的请求头字段含义：
Host：接受请求的服务器地址，IP：端口号  或 域名；
User-Agent：发送请求的应用程序名称；
Connection：指定连接相关属性，如：Connection: keep-alive 长连接；
Accept：浏览器可接受的MIME类型；
Accept-Charset: 通知服务端可以发送的编码格式；
Accept-Encoding: 通知服务端可以发送的数据格式；
Accept-Language: 通知服务端可以发送的语言；
Authorization：授权信息，通常出现在对服务器发送的WWW-Authenticate头的应答中。
Cookie：客户机通过这个头可以向服务器带数据，这是最重要的请求头信息之一。

### HTTP响应报文
![http响应报文格式](https://github.com/SweetyPeng/Pilgrim/blob/master/assets/http响应报文格式.png?raw=true)

#### 状态码
1xx——指示信息，表示请求已接收，继续处理
2xx——成功，表示请求已被成功接收、理解、接受
3xx——重定向，要完成请求必须进行更进一步的操作
4xx——客户端错误，请求有语法错误或请求无法实现
5xx——服务器端错误，服务器未能实现合法的请求

常见：
200——响应成功，所请求的资源发送回客户端；
301--永久重定向，搜索引擎将删除源地址，保留重定向地址；
304——自从上次请求后，请求的网页未修改过，缓存文件未过期，无需从服务端获取，请客户端使用本地缓存
400——客户端请求有错
401——请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用。
403——服务端收到请求，但是拒绝提供服务，认证失败；
500——服务器内部错误
503——服务不可用

## Cookie
cookie是浏览器的一种本地存储方式，一般用来帮助客户端和服务端通信的，常用来进行身份校验，结合服务端的session使用。
```text
场景如下（简述）：
在登陆页面，用户登陆了
此时，服务端会生成一个session，session中有对于用户的信息（如用户名、密码等）
然后会有一个sessionid（相当于是服务端的这个session对应的key）
然后服务端在登录页面中写入cookie，值就是:jsessionid=xxx
然后浏览器本地就有这个cookie了，以后访问同域名下的页面时，自动带上cookie，自动检验，在有效时间内无需二次登陆。
上述就是cookie的常用场景简述（实际情况下得考虑更多因素）
```
![cookie](https://github.com/SweetyPeng/Pilgrim/blob/master/assets/cookie.jpg?raw=true)
