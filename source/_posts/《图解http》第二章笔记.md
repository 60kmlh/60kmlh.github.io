---
title: 《图解http》第二章笔记
date: 2017-09-16
tags: ['http']
categories: ['笔记']
---
## 简单的http协议
两台计算机使用HTTP协议进行通信时，在一条通信线路上必定有一端是客户端，另一端则是客户端。

### 下面是客户端发给某个HTTP客户端的请求报文中的内容。
````http
GET /index.html HTTP/1.1
HOST: hackr.jp
````
GET表示请求访问服务器的类型，称为方法（method）。

随后的/index.html指明请求访问的资源对象，叫做请求URI（request-URI）。

最后的HTTP/1.1表明HTTP的版本号。

请求报文由请求方法、请求URI、协议版本、可选的请求首部字段和内容实体构成。
### 接收到请求的服务器，会将请求内容的处理结果以响应的形式返回。
````http
HTTP/1.1 200 OK
Date: Tue, 10 Jul 2012 06:50:15 GMT
Content-Length: 362
Content-Type: text/html

<html>
....
````
开头的HTTP/1.1代表服务端对应的HTTP版本。

紧接着的200 OK代表请求的处理结果的状态码（status code）和原因短语（reason-phrase）。

下一行显示创建响应的日期时间、内容长度、内容类型，是首部字段（header field）内的一个属性。

接着以一空行分隔，之后的内容称为资源内容的主体（entity body）。

响应报文基本上由协议版本、状态码、解释状态码的原因短语、可选的响应首部字段已经实体主体构成。
### HTTP是一种不保存状态，即无状态（stateless）协议。
http协议自身不对请求和响应之间的通信状态进行保存。
### 请求URI定位资源
请求指定URI的方式
1. URI为完整的URI
````http
GET http://hackr.jp/index.htm HTTP/1.1
````
2. 在首部字段Host中写明网络域名或IP地址
````http
GET /index.htm HTTP/1.1
Host: hackr.jp
````
### 告知服务器意图的HTTP方法
* GET：获取资源
    
    用来请求访问已被URI识别的资源。指定的资源经服务端解析后返回响应的内容。
* POST：传输主体实体

    用来传输实体的主体。
* PUT：传输文件
    
    要求在请求报文的主体包含文件内容，然后保存到URI指定的位置。

    鉴于HTTP/1.1的PUT方法不带验证机制，任何人都可以上传文件，因此存在安全性问题,一般不使用。
* HEAD：获得报文首部

    HEAD方法和GET方法一样，这是不返回报文的主体部分。

    用于确认URI的有效性及资源更新的日期时间等。
* DELETE：删除文件

    DELETE方法按请求的URI删除指定的资源。

    但是DELETE方法和PUT方法一样不带验证机制，一般也不使用DELETE方法。
* OPTIONS：查询支持的方法

    用来查询针对请求URI指定的资源支持的方法。
* TRACE：追踪路径

    TRACE方法是让Web服务器将之前请求通信环回给客户端的方法。可以查询发送出去的请求是怎样被加工或者篡改的。
* CONNECT：要求用隧道协议连接代理
    
    该方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行TCP通信。

    主要使用SSL（Secure Sockets Layer，安全套接层）和TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输。
### 持久连接节省通信量
1. 持久连接
在初始版本的HTTP协议中，每进行一次HTTP通信就要断开一次TCP连接，增加通信量的开销。

为解决上述TCP连接的问题，HTTP/1.1和一部分HTTP/1.0提出了持久化的连接（HTTP Persistent Connections），也叫做HTTP keep-alive或HTTP connection reuse。

持久化连接的特点是，只要任意一方没有明确提出断开连接，则保持TCP连接状态。

好处是减少了TCP连接的重复建立和断开所造成的额外开销，减轻服务器端的负载。减少开销的那部分时间，使得Web页面的显示速度提高。

在HTTP/1.1中，默认所有连接都是持久连接。

持久连接使得多数请求以管线化（pipeling）方式发送成为可能，可以并行发送请求。
### 使用cookie的状态管理
cookie技术通过在请求和响应的报文中写入cookie信息来控制服务端状态。

cookie会根据从服务器发送的响应报文内的一个叫做Set-Cookie的首部字段信息，通知客户端保存cookie。下次客户端再往服务端发送请求时，客户端会自动在请求报文中加入cookie值后发送出去。

服务端接收到客户端发送过来的cookie后会去检查究竟是哪个客户端发来的连接请求，对比服务器上的记录，最后得到之前的状态信息。
### If-Modified-Since
If-Modified-Since是标准的HTTP请求头标签，在发送HTTP请求时，把浏览器端缓存页面的最后修改时间一起发到服务器去，服务器会把这个时间与服务器上实际文件的最后修改时间进行比较。

如果时间一致，那么返回HTTP状态码304（不返回文件内容），客户端接到之后，就直接把本地缓存文件显示到浏览器中。

如果时间不一致，就返回HTTP状态码200和新的文件内容，客户端接到之后，会丢弃旧文件，把新文件缓存起来，并显示到浏览器中。
### 参考资料
* [https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Modified-Since](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Modified-Since)
* [http://www.cnblogs.com/zh2000g/archive/2010/03/22/1692002.html](http://www.cnblogs.com/zh2000g/archive/2010/03/22/1692002.html)
