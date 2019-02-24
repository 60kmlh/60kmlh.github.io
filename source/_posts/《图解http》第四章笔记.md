---
title: 《图解http》第四章笔记
date: 2017-09-23
tags: ['http']
categories: ['笔记']
---
## 返回结果的HTTP状态码
### 状态码告知从服务器端返回的请求结果
状态码的职责是当客户端向服务器端发送请求时，描述返回的请求结果。

状态码由3位数字和原因短语组成。数字中的第一位指明响应类别，后两位无分类。

状态码的类别
状态码 | 类别 | 原因短语
------:|------|------
1xx   | Informational（信息性状态码） | 接收的请求这种处理
2xx   | Success（成功状态码） | 请求正常，处理完毕
3xx   | Redirection（重定向状态码） | 需要进行附加操作以完成请求
4xx   | Client Error（客户端错误性状态码） | 服务器无法处理请求
4xx   | Server Error（服务端错误状态码） | 服务器处理请求出错
### 2xx 成功
* 200 OK
    表示从客户端发来的请求被服务端正常处理了
* 204 No Content
    表示服务器接收的请求处理成功，但在返回的响应报文中不含实体的主体部分。
* 206 Partial Content
    表示客户端进行了范围请求，而服务器成功执行了这部分GET请求。

    响应报文中由Content-Range指定范围的实体内容。
### 3xx 重定向
* 301 Move Permanently
    永久性重定向，表示请求的资源已经被分配了新的URI，以后应使用资源现在所指的URI。
* 302 Found
    临时性重定向，表示请求的资源已经被分配了新的URI，希望用户（本次）能够使用新的URI访问。
* 303 See Other
    表示由于请求对应的资源存在着另一个URI，应使用GET方法定向获取请求的资源。和302的区别是，明确表示使用GET获取。
* 304 Not Modified
    表示客户端发送附带条件的请求时，服务端允许请求访问资源，但为满足条件的情况。

    304状态码返回时，不包含任何响应的主体部分。
    
    附带的条件值采用GET方法的请求报文中包含If-Match、If-Modified-Since、If-None-Match、If-Range、If-Unmodified-Since中的任意首部。
* 307 Temporary Redirect
    表示临时重定向
### 4xx 错误
* 400 Bad Request
    表示请求报文中发生语法错误。
* 401 Unauthorized
    表示发送的请求需要有通过HTTP认证（BASIC认证， DIFEST认证）的认证信息。

    返回含有401的响应必须包含一个适用于被请求资源的WWW-Authenticate首部用以质询（challenge）用户信息。
* 403 Fobbiden
    标明对请求资源的访问被服务器拒绝了。
* 404 Not Found
    表明服务器上无法找到请求的资源。
### 5xx 服务器错误
* 500 Internal Server Error
    表明服务端在执行请求是发生了错误。
* 503 Server Unavailable
    表明服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

    如果事先得知解除以上状况需要的时间，最好写入Retry-After首部字段再返回给客户端。
