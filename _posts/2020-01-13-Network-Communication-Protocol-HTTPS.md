---
layout: post
title: 网络通信基础（二）HTTP和HTTPS通信协议
tags: 网络通信基础  
---

<!-- TOC -->

- [HTTP/HTTPS请求的组成部分](#httphttps请求的组成部分)
    - [相关概念](#相关概念)
        - [URI](#uri)
        - [URL](#url)
        - [DNS](#dns)
        - [CDN](#cdn)
- [HTTPS](#https)
- [HTTP/HTTPS请求包含的内容](#httphttps请求包含的内容)
    - [请求方式（这里只列举最常用的四种）](#请求方式这里只列举最常用的四种)
    - [协议类型和版本](#协议类型和版本)
    - [Cookie](#cookie)
    - [其他的一些参数](#其他的一些参数)
- [为什么需要HTTPS？](#为什么需要https)
    - [HTTP 特点](#http-特点)
    - [HTTPS 的设计思想过程](#https-的设计思想过程)
        - [（1）引入对称加密，对传输内容进行加解密](#1引入对称加密对传输内容进行加解密)
        - [（2）使用非对称加密](#2使用非对称加密)
        - [（3）使用非对称加密来生成对称加密的密钥](#3使用非对称加密来生成对称加密的密钥)
        - [（4）引入受信任的第三方机构签发证书](#4引入受信任的第三方机构签发证书)
- [浏览器如何验证证书真伪？？](#浏览器如何验证证书真伪)

<!-- /TOC -->
# HTTP/HTTPS请求的组成部分
## 相关概念
### URI
URI的全称是（Uniform Resource Identifier），中文名称是统一资源标识符，使用它就能够唯一地标记互联网上资源。

### URL
URL的全称是（Uniform Resource Locator），中文名称是统一资源定位符，也就是我们俗称的网址，它实际上是 URI 的一个子集。
URI 不仅包括 URL，还包括 URN（统一资源名称），它们之间的关系如下
![](/images/posts/myBlog/2020-01-13-Network-Communication-Protocol-HTTPS-01.png)

### DNS
我们前面介绍了 `HTTP` 数据包传输过程中的 `TCP` 头信息会加入 `ip` 地址，但是我们访问服务器是通过域名的形式访问的，当浏览器键入 `www.google.com` 时，如何知道我们需要访问的目标 `ip` （门牌号）的？ 所以我们需要有一个解析域名的成 `IP` 地址的方法，这个协议就是 `DNS` 协议。
`DNS` 的全称是域名系统（`Domain Name System`，缩写：`DNS`），它作为将域名和 IP 地址相互映射的一个分布式数据库，能够使人更方便地访问互联网。

### CDN
`CDN` 的全称是 `Content Delivery Network`，即内容分发网络，它应用了 `HTTP` 协议里的缓存和代理技术，代替源站响应客户端的请求。`CDN` 是构建在现有网络基础之上的网络，它依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率。`CDN` 的关键技术主要有内容存储和分发技术。

打比方说你在京东下单买书，请求的收货地址是广州，那么京东就会有先选择离你最近的广州的分仓库里调货，而不选择通过远在北京的仓库调货。

# HTTPS
`HTTP` 一般是明文传输，很容易被攻击者窃取重要信息，鉴于此，`HTTPS` 应运而生。`HTTPS` 的全称为 （`Hyper Text Transfer Protocol over SecureSocket Layer`），全称有点长，`HTTPS` 和 `HTTP `有很大的不同在于 `HTTPS` 是以安全为目标的 `HTTP` 通道，在 `HTTP` 的基础上通过传输加密和身份认证保证了传输过程的安全性。`HTTPS` 在 `HTTP` 的基础上增加了 `SSL` 层，也就是说 `HTTPS` = `HTTP` + `SSL`

# HTTP/HTTPS请求包含的内容
这个是抓取的Bilibili的一个接口的结果
![](/images/posts/myBlog/2020-01-13-Network-Communication-Protocol-HTTPS-02.png)

具体的参数解析如下：
``` properties
POST https://api.bilibili.com/x/click-interface/click/web/h5 HTTP/1.1  //请求方式  请求URL HTTP协议版本号
Host: api.bilibili.com  //请求域名
Connection: keep-alive  //连接方式
Content-Length: 138     //数据长度
Sec-Fetch-Mode: cors    
Origin: https://www.bilibili.com    //站点信息
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.75 Safari/537.36  //请求客户端信息
Content-type: application/x-www-form-urlencoded     //参数类型
Accept: */*
Sec-Fetch-Site: same-site
Referer: https://www.bilibili.com/bangumi/play/ss28628/?spm_id_from=333.851.b_62696c695f7265706f72745f616e696d65.40     //告诉服务器从哪个页面过来的，防止劫持
Accept-Encoding: gzip, deflate, br  //编码格式
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8    //语言信息Local
Cookie: _uuid=E3B4B31B-37C6-544F-8554-E45DC12C0CAB14482infoc //Cookie信息

//以下是Post的内容，长度就是上面的Content-Length
aid=70883403&cid=122816570&part=1&mid=219682899&lv=4&ftime=1576422353&stime=1578842175&jsonp=jsonp&type=4&sub_type=1&sid=28628&epid=286280
```

## 请求方式（这里只列举最常用的四种）
1.` GET` ：向特定的路径资源发出请求，数据暴露在url中
2. `POST` ：向指定路径资源提交数据进行处理请求（一般用于上传表单或者文件），数据包含在请求体中
3. `PUT` ：从客户端向服务器传送的数据取代指定的文档的内容 
4. `DELETE` ：请求服务器删除指定的页面

## 协议类型和版本
- https:// : 代表着使用的协议是 `HTTPS` 协议
- HHTP/1.1 ： 使用的HTTP协议版本是1.1

## Cookie
因为 `HTTP` 是无状态的连接，服务器需要知道两次请求是同一个客户端发起的，比如用户已经登录的状态，需要维持这个状态，就需要用到 `Cookie` 信息，让服务端进行登录状态的鉴权。

## 其他的一些参数
- `Connection` ： keep-alive :保持连接，在一定时间内重新发起 `HTTP` 请求并不需要重新发起3次握手
- `User-Agent` : 请求客户端信息,包含系统、浏览器的版本等信息
- `Accept-Encoding` : 编码方式


# 为什么需要HTTPS？
要回答这个问题，需要先明白 `HTTP` 协议的特点：
## HTTP 特点
    1. 无状态：协议对客户端没有状态存储，对事物处理没有“记忆”能力，比如访问一个网站需要反复进行登录操作
    2. 无连接：HTTP/1.1之前，由于无状态特点，每次请求需要通过TCP三次握手四次挥手，和服务器重新建立连接。比如某个客户机在短时间多次请求同一个资源，服务器并不能区别是否已经响应过用户的请求，所以每次需要重新响应请求，需要耗费不必要的时间和流量。
    3. 基于请求和响应：基本的特性，由客户端发起请求，服务端响应
    4. 简单快速、灵活
    5. 通信使用明文、请求和响应不会对通信方进行确认、无法保护数据的完整性

使用 `HTTP` 协议可以很简单的实现通信，但是，由于其是使用明文传输的，非常容易被黑客进行劫持，所以就引入了加密协议 `SSL/TLS` ，也就是我们所说的 `HTTPS` = `HTTP` + `SSL/TLS` 

## HTTPS 的设计思想过程
### （1）引入对称加密，对传输内容进行加解密
![](/images/posts/myBlog/2020-01-13-Network-Communication-Protocol-HTTPS-03.png)

在使用对称加密前，需要双方约定加密的Key，这样就有可能被中间人被中间人截取到约定的加密Key，就算在约定加密的key前没有没被截取到密钥，也有以下的缺点
- （1）不同的客户端、服务器数量庞大，所以双方都需要维护大量的密钥，维护成本很高

- （2）因每个客户端、服务器的安全级别不同，密钥极易泄露

### （2）使用非对称加密
![](/images/posts/myBlog/2020-01-13-Network-Communication-Protocol-HTTPS-04.png)

缺点：公钥是公开的（也就是黑客也会有公钥），所以私钥加密的信息，如果被黑客截获，其可以使用公钥进行解密，获取其中的内容


### （3）使用非对称加密来生成对称加密的密钥
![](/images/posts/myBlog/2020-01-13-Network-Communication-Protocol-HTTPS-05.png)

遇到的问题：

- （1）客户端如何获得公钥
- （2）如何确认服务器是真实的而不是黑客
### （4）引入受信任的第三方机构签发证书
为了解决公钥获取的问题，防止公钥的获取被黑客获取，引入了权威的第三方机构来生成数字证书。
把服务器的域名，公钥等信息提交到权威的CA机构申请证书，CA机构本身也有一对公钥和私钥，CA证书使用秘钥对申请的信息进行加密，生成证书，而将公钥公开给客户端解密证书，如下图：
![](/images/posts/myBlog/2020-01-13-Network-Communication-Protocol-HTTPS-06.png)


# 浏览器如何验证证书真伪？？
- （1）首先浏览器读取证书中的证书所有者、有效期等信息进行一一校验
- （2）浏览器开始查找操作系统中已内置的受信任的证书发布机构CA，与服务器发来的证书中的- 颁发者CA比对，用于校验证书是否为合法机构颁发 
- （3）如果找不到，浏览器就会报错，说明服务器发来的证书是不可信任的（只是报错，浏览器没有绿色的图标，并不影响通信）。
- （4）如果找到，那么浏览器就会从操作系统中取出 颁发者CA 的公钥，然后对服务器发来的证书里面的签名进行解密
- （5）浏览器使用相同的hash算法计算出服务器发来的证书的hash值，将这个计算的hash值与证书中签名做对比
- （6）对比结果一致，则证明服务器发来的证书合法，没有被冒充
- （7）此时浏览器就可以读取证书中的公钥，用于后续加密了
