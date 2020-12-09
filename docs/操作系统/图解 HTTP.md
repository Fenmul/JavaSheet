# 图解 HTTP

> 说明：图解 HTTP 读书笔记

### TCP/IP
![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116126610-2707afb9-c0b0-4c6f-909b-9beb9ec837bc.png#align=left&display=inline&height=611&margin=%5Bobject%20Object%5D&originHeight=611&originWidth=838&size=0&status=done&style=none&width=838)
TCP/IP 协议族 包含很多协议: HTTP,FTP,TCP,IP 等


请求 www.example.com
![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116140393-02b41967-722f-4ced-8c73-891f7a35b815.png#align=left&display=inline&height=774&margin=%5Bobject%20Object%5D&originHeight=774&originWidth=960&size=0&status=done&style=none&width=960)


#### URI 和 URL


- URI：URI 用字符串标识某一互联网资源
- URL: 字符换标识资源**地址**



#### IP 协议


位于网络层


域名通过 DNS 服务映射到 IP 地址（域名解析）之后访问目标网站，DNS 的作用类似于一个 Map 集合 Key-Value 的形式去匹配域名和 IP 地址。


- IP： IP 协议把各种数据包传送给对方。
- IP 地址：指定节点被分配到的地址
- MAC 地址：网卡所属的固定地址，基本不会更改。



**IP 间的通讯依赖 MAC地址**，中转过程会通过 MAC 地址寻找到下一站中转设备。采用 ARP 协议解析 IP 地址反查出 MAC 地址
![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116155127-7087769d-da12-490f-ae7f-42709a8002a4.png#align=left&display=inline&height=696&margin=%5Bobject%20Object%5D&originHeight=696&originWidth=850&size=0&status=done&style=none&width=850)


#### TCP


位于传输层，提供**可靠的字节流服务**，为了方便传输，将文件分割成报文管理。


##### 三次握手


握手过程中使用了 TCP 的标志（flag）—— SYN（synchronize）和 ACK（acknowledgement）
![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116161734-c3310274-37b0-4124-ab28-2537455d677d.png#align=left&display=inline&height=441&margin=%5Bobject%20Object%5D&originHeight=441&originWidth=824&size=0&status=done&style=none&width=824)


三次握手：接收和发送双方均确认接收和发送是正常的


1. 接收端确认了发送端的发送正常和接收端的接收正常
1. 发送端确认了自身发送正常，接收正常，接收端发送正常
1. 接收端确认自身接收正常



##### 四次握手


任何一方都可以在传送结束发送断开通知，待对方确认进入半关闭状态，当对方也没有消息发送的时候，发出释放连接通知，自身关闭，对方确认完全关闭。


1. 发送端发送 FIN 表明要断开连接
1. 接收端发送 ACK
1. 接收端发送 FIN 表明也要断开
1. 发送端发送 ACK 确认断开



### HTTP 报文


#### 持久化


HTTP 1.1 支持持久连接
![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116168314-32d2b949-5d80-4925-8f7e-3f2f786a9981.png#align=left&display=inline&height=631&margin=%5Bobject%20Object%5D&originHeight=631&originWidth=827&size=0&status=done&style=none&width=827)


管线化：并行发送多个请求的基础
![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116173146-5d24e82d-ce01-4c11-aa96-d57cd4002765.png#align=left&display=inline&height=354&margin=%5Bobject%20Object%5D&originHeight=354&originWidth=771&size=0&status=done&style=none&width=771)


#### Cookie


HTTP 协议是无状态的，因此可以借助 Cookie 辨别请求的身份。Cookie 在服务器中生成，通过响应报文让客户端保存，每次请求都携带上 Cookie。


1. 请求报文



```java
GET /reader/ HTTP/1.1
Host: hackr.jp
*首部字段内没有Cookie的相关信息
```


2. 响应报文（服务器生成 Cookie 信息）



```
HTTP/1.1 200 OK
Date: Thu, 12 Jul 2012 07:12:20 GMT
Server: Apache
＜Set-Cookie: sid=1342077140226724; path=/; expires=Wed,
10-Oct-12 07:12:20 GMT＞
Content-Type: text/plain; charset=UTF-8
```


3. 请求报文（携带 Cookie）



```
GET /image/ HTTP/1.1
Host: hackr.jp
Cookie: sid=1342077140226724
```


#### 报文和实体


- 报文：HTTP 通信的基本单位
- 实体：作为请求或响应的有效载荷数据（补充项）被传输，其内容由实

体首部和实体主体组成。
- 通常报文主题等于实体主题，只有在传输过程中进行编码操作，实体主体才会发生变化。**编码可以有效的提升传输速率，但同时也会消耗更多的 CPU 资源**



#### 压缩编码


压缩传输内容编码：应用在实体内容上的编码格式，保持实体信息原样压缩。


页面资源耗时过长，使用内容编码压缩资源，在 springboot 中 gzip 内容压缩可以通过如下配置实现


```yaml
server:
  compression:
    enabled: true
    mime-types: text/html, text/css, text/javascript, application/javascript
    min-response-size: 1024
```


#### 内容协商


使用场景：同一个 URI 的 Web 页面显示不同版本页面（中文和英文）


概念：参考请求头中  Accept，Accept-Charset，Accept-Encoding，Accept-Language，Content-Language 作为判断的基准


- 服务器驱动协商
- 客户端驱动协商
- 透明协商



#### MIME 多用途因特网邮件扩展


浏览器附件上传也是利用 MIME 来描述标记数据类型，MIME 扩展中我们会使用一种 Multipart （多部分对象集合）来容纳多份不同类型的数据。


在 HTTP 报文中使用多部分对象集合时，需要在首部字段里加上
Content-type。


- multipart/form-data：文件上传
- multipart/byteranges：想要包含多个范围的内容
- multipart/form-data：boundary 字符串划分各类实体



![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116248189-24aa58fb-1448-410d-ba32-11f6e429e8cb.png#align=left&display=inline&height=1075&margin=%5Bobject%20Object%5D&originHeight=1075&originWidth=2043&size=0&status=done&style=none&width=2043)


#### byteranges


中断恢复下载: 要实现该功能需要指定下载的实体范围。像这样，指定范围发送的请求叫做范围请求（Range Request）。


对一份 10000 字节大小的资源，如果使用范围请求，可以只请求
5001~10000 字节内的资源。


执行请求的时候，Range 指定资源的 byte 范围


- 5001~10000 字节 `Range: bytes=5001-10000`
- 5001向后 `Range: bytes=5001-`
- 从一开始到 3000 字节和 5000~7000 字节的多重范围 `Range: bytes=-3000, 5000-7000`



### 状态码的类别
| 类别 | 原因短语 | 原因 |
| --- | --- | --- |
| 1XX | Informational（信息性状态码） | 接收的请求正在处理 |
| 2XX | Success（成功状态码） | 请求正常处理完毕 |
| 3XX | Redirection（重定向状态码） | 需要进行附加操作以完成请求 |
| 4XX | Client Error（客户端错误状态码） | 服务器无法处理请求 |
| 5XX | Server Error（服务器错误状态码） | 服务器处理请求出错 |



#### 2XX 请求成功
| 代码 | 原因短语 | 原因 | 使用场景 |
| --- | --- | --- | --- |
| 200 | OK | 请求正常处理 |  |
| 204 | No Content | 请求处理成功但没有可返回资源 | 只需要客户端向服务端发送，客户端不需要更新信息 |
| 206 | Partial Content | 范围请求 | 对于资源的某一部分请求 |



#### 3XX 重定向
| 代码 | 原因短语 | 原因 | 使用场景 |
| --- | --- | --- | --- |
| 301 | Moved Permanently | 永久性重定向 | 资源已被分配新的 URI |
| 302 | Found | 临时性重定向 | 类似于临时的 301 只需要客户端向服务端发送，客户端不需要更新信息 |
| 303 | See Other | 使用 GET |  |
| 方法定向获取请求的资源 | 对于资源的某一部分请求**客户端应当采用 GET 方法获取资源** |  |  |
| 304 | Not Modified | 服务端允许请求，但未满足条件 | **与重定向无关**，GET 请求包含特定的头信息 `If-Match`等 |
| 307 | Temporary Redirect | 临时重定向 | 与 302 相同，但是不允许 POST 变为 GET |



> 尽管 302 也不允许由 POST 变换成 GET 但是处理响应的时候不同浏览器会出现不同的处理



304 详细解析
![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116260349-8b30e0fe-30ff-49b3-b59c-78e8998a3ffe.png#align=left&display=inline&height=930&margin=%5Bobject%20Object%5D&originHeight=930&originWidth=1275&size=0&status=done&style=none&width=1275)


当客户端缓存了目标资源但不确定该缓存资源是否是最新版本的时候, 就会发送一个条件请求。在进行条件请求时,客户端会提供给服务器一个If-Modified-Since请求头,其值为服务器上次返回响应头中Last-Modified值。


服务器会读取到这个请求头中的值,判断出客户端缓存的资源是否是最新的,如果是的话,服务器就会返回 HTTP/304 Not Modified 响应头, 但没有响应体.客户端收到 304 响应后,就会从本地缓存中读取对应的资源。 所以：当访问资源出现 304 访问的情况下其实就是先在本地缓存了访问的资源。


#### 4XX 客户端错误
| 代码 | 原因短语 | 原因 | 使用场景 |
| --- | --- | --- | --- |
| 400 | Bad Request | 请求报文中存在语法错误 | 服务器无法理解客户端请求 |
| 401 | Unauthorized | 请求需要通过 HTTP 认证 | 弹出认证的窗口 |
| 403 | Forbidden | 请求资源被拒绝 | 服务器不需要给出详细说明，可以在 实体主体中描述原因 |
| 404 | Not Found | 服务器无法找到资源 | 服务器拒绝请求，不想说明理由 |
| 405 | Method Not Allowed | 不支持的 HTTP 方法 | 可以通过 Allow 参数通知客户端允许的方法 |



#### 5XX 服务器错误
| 代码 | 原因短语 | 原因 | 使用场景 |
| --- | --- | --- | --- |
| 500 | Internal Server Error | 服务器内部资源故障 | 应用存在 BUG |
| 503 | Service Unavailable | 超负载或正在进行停机维护 | 可以在请求头中的 RetryAfter 字段返回给客户端 |



### WEB


#### 代理、网关和隧道


- 代理：代理是复制转发的应用程序，连接客户端和服务器，**协议不变**
   - 缓存代理：代理转发时会将预先将资源副本保存在代理服务器，请求就不会从源服务器获取资源
   - 透明代理：转发请求不对报文做任何处理，反之怎么成为非透明代理



![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116269352-45a6da02-423c-4701-b57f-e595cb3a1588.png#align=left&display=inline&height=265&margin=%5Bobject%20Object%5D&originHeight=265&originWidth=823&size=0&status=done&style=none&width=823)


每次通过代理服务器转发请求或响应时，会追加写入 Via 首
部信息


- 网关

网关的工作机制和代理十分相似，而网关能使通信线路上的服务器提

供非 HTTP 协议服务。此外网关可以在通信线路上加密确保连接的安全性。
- 隧道

通过隧道的传输，可以和远距离的服务器安全通信。隧道本

身是透明的，客户端不用在意隧道的存在



### HTTP 头部


##### 请求首部字段


Accept：通知服务器，用户代理能够处理的媒体类型和优先级。具体使用 type/subtype 指定媒体类型


`Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8` ：若想要给显示的媒体类型增加优先级，则使用 q= 来额外表示权重值
1，用分号（;）进行分隔。权重值 q 的范围是 0~1（可精确到小数点
后 3 位），且 1 为最大值。不指定权重 q 值时，默认权重为 q=1.0。


Accept-Encoding：告知服务器用户代理支持的内容编码和优先级顺序 `Accept-Encoding: gzip, deflate`


##### 实体首部字段


Content-Encoding：告知客户端服务器对实体的主体部分选用的内容编码方式。


`Content-Encoding: gzip` 服务器采用 gzip 的方式进行内容编码压缩


`Content-Length: 15000` 实体主题部分为 15000 字节


Content-Type: 说明实体主题内对象的媒体类型。和首部字段 Accept 相同采用


`Content-Type: text/html; charset=UTF-8`


DNT：拒绝个人信息被收集，0：同意，1：拒绝


##### Cookie 相关字段


Set-Cookie：开始状态管理所使用的Cookie信息 响应首部字段


`Set-Cookie: status=enable; expires=Tue, 05 Jul 2011 07:26:31 GMT; path=/; domain=.hackr.jp;`


expires 属性：设置有效期，并且一旦 Cookie 由服务器端发送后，服务器端就不可以显示的删除 Cookie ，但是可以通过覆盖过期 Cookie 的实质性删除操作


secure 属性：只有在 HTTPS 安全连接才可以发送 Cookie


HttpOnly 属性：js 脚本无法获得 Cookie，防止信息窃取


Cookie：服务器接收到的Cookie信息


### HTTPS


概念：用 SSL建立安全通信线路之后，就可以在这条线路上进行 HTTP 通信了。与 SSL组合使用的 HTTP 被称为 HTTPS（HTTP Secure，超文本传输安全协议）或 HTTP over SS


**HTTP+ 加密 + 认证 + 完整性保护 = HTTPS**
![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116278321-27c75d24-1254-4815-b1ca-422f497962a4.png#align=left&display=inline&height=302&margin=%5Bobject%20Object%5D&originHeight=302&originWidth=689&size=0&status=done&style=none&width=689)


#### 加密


SSL采用一种叫做公开密钥加密（Public-key cryptography）的加密处理方式。


公开密钥加密方式很好地解决了共享密钥加密的困难：使用公开密钥加密方式，发送密文的一方使用对方的公开密钥进行加密处理，对方收到被加密的信息后，再使用自己的私有密钥进行解密。利用这种方式，不需要发送用来解密的私有密钥，也不必担心密钥被攻击者窃听而盗走。
![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116283739-4c6bf1e4-d604-44af-96a4-bc15bd2525f2.png#align=left&display=inline&height=583&margin=%5Bobject%20Object%5D&originHeight=583&originWidth=800&size=0&status=done&style=none&width=800)


HTTPS 采用共享和公开混合加密，共享的速度要比公开的方式更快


Session 和 Cookie


![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116289103-d7c1d26e-48aa-4b82-96a0-07edd30ef564.png#align=left&display=inline&height=399&margin=%5Bobject%20Object%5D&originHeight=399&originWidth=838&size=0&status=done&style=none&width=838)


#### WebSocket


为了实现 WebSocket 通信，需要用到 HTTP 的 Upgrade 首部字 段，告知服务器通信协议发生改变，以达到握手的目的。


```
GET /chat HTTP/1.1 
Host: server.example.com 
Upgrade: websocket 
Connection: Upgrade 
# 握手过程中必不可少的键值对
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ== 
Origin: http://example.com 
# 记录协议
Sec-WebSocket-Protocol: chat, superchat 
Sec-WebSocket-Version: 13
```


响应端 返回  101 Switching Protocols


![](https://cdn.nlark.com/yuque/0/2020/png/1471883/1590116294837-13c2d529-156c-4695-8295-bbe6e1bd85be.png#align=left&display=inline&height=1012&margin=%5Bobject%20Object%5D&originHeight=1012&originWidth=1620&size=0&status=done&style=none&width=1620)


##### CGI


CGI（Common Gateway Interface，通用网关接口）是指 Web 服务器在 接收到客户端发送过来的请求后转发给程序的一组机制。


由于每次接到请求，程序都要跟着启动一次。因此一旦访问量过大，Web 服务器要承担相当大的负载。而 Servlet 运行在与 Web 服务器相同的进程中，因此受到的负载较小。Servlet 的运行环境叫做 Web 容器或 Servlet 容器。


Servlet 也就是 CGI 的 Java 轻量优化版本。
