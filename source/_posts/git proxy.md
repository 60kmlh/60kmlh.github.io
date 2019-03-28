---
title: Git Proxy
date: 2017-07-16
tags: ['git', 'proxy']
categories: ['git']
---
## git设置代理
### http代理
````bash
git config --global https.proxy http://127.0.0.1:1080

git config --global https.proxy https://127.0.0.1:1080

#取消代理
git config --global --unset http.proxy

git config --global --unset https.proxy
````
<!--more-->
#### 针对github的代理
````bash
#只对github.com进行代理
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080

#取消代理
git config --global --unset http.https://github.com.proxy
````
### ssh代理
对于使用git@协议的，可以配置socks5代理
在~/.ssh/config 文件后面添加几行，没有可以新建一个
````bash
Host github.com
ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p
````
* windows下

因为这个bash是不带netcat的，也就找到不到nc命令。
在win10上，有的msysgit版本集成了connect工具，所以在windows上，可以把ssh的config文件设置为：
````bash
Host github.com
ProxyCommand connect -S 127.0.0.1:1080 %h %p
````
### 参考资料
[git 设置和取消代理](https://gist.github.com/laispace/666dd7b27e9116faece6)

[Tutorial: how to use git through a proxy](http://cms-sw.github.io/tutorial-proxy.html)

[GitConfigHttpProxy](https://gist.github.com/evantoli/f8c23a37eb3558ab8765)

