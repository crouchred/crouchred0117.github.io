---
layout: post
title: HTTP中GET和POST的区别
description: HTTP中GET和POST的区别
category: 技术
tags: [网络]
---

# HTTP中GET和POST的区别

## 1. 缘起
老实说我对get和post的区别已经完全糊涂了。糊涂的原因大概因为遇到一些奇怪的情况：
  
1. 看到很多人说，get用来获取资源，post用来新建资源。但是根据最近在做爬虫的经验，登陆请求一般都用的是post，可是登陆的时候新建了什么资源呢？况且返回的状态码都是200，怎么看没有新建一个资源吧。
2. 在学习python的requests库的过程中，在v0.5.0之前post请求返回的status_code是201，但是v0.5.0开始全部(即使是上传文件)换成了200，但是大神在commit里面并没有说明原因。我自己请求了一下网站返回的结果确实是200。大神在干啥。。

这两个案例都让我很迷茫，如果post是用来新建资源的，那么首先登陆不应该用post，其次post请求返回的结果应该是201，于是啃哧啃哧的开始翻网上的文章。


## 2. Restful架构
后来才知道，如果要解释上述的两个问题，其实一句话就说明白了，**get用来获取资源，post用来新建资源，这是在restful架构下的设计**，这个架构他只是一种风格，而非一种规定。 于是回到刚才的问题。

1. 登陆请求是restful架构的吗？显然不是，restful的url是要对应一个资源的,`http://xxx.com/login`，login显然不是一个资源，而是一个动作。另外这个请求也没有创建一个资源。
2. requests用来测试的网站`https://httpbin.org`真的只是一个用来测试的网站，并没有说遵从restful架构，他想返回什么就返回什么。

哦原来如此，现在我们可以抛开Rest来谈区别了。

## 3. GET和POST的区别
Restful是在http之上的一层概念，那么我们回到http的本来面目，get和post的区别到底什么呢？比如为什么一般登陆都是用post请求呢？总结了一些二手的观点。  

### 3.1 直观区别
[rfc2616](https://tools.ietf.org/html/rfc2616#section-9.3) 中说到：**The GET method means retrieve whatever information ([...]) is identified by the Request-URI** 。也就是说：

+ GET请求只从URL的参数中获取数据
+ POST可以同时从参数和body中获取数据（POST从参数中获取数据好不好另说，可见[这篇文章](https://stackoverflow.com/questions/611906/http-post-with-url-query-parameters-good-idea-or-not)）。  

一个比较有意思的地方是，request库在v0.5.0之前是不支持post传递params参数的，详见[80d860d](https://github.com/requests/requests/commit/f31ade335de9df096ba3c67c9507dd4eb32667a8), [f31ade3](https://github.com/requests/requests/commit/f31ade335de9df096ba3c67c9507dd4eb32667a8), 再后来似乎get也让传body了？后面看了再来补充。

### 3.2衍生区别
正是由于上面所说的这个区别，导致GET和POST在应用过程中体现出一些不同： 

1. **浏览器对URL的长度有限制，所以GET不能代替POST发送大量数据**。RFC2616中对URL长度没有规定，但是浏览器比如ie和chrome一般都会限制。
2. **GET方法应该是幂等的**。幂等啥意思，就是说多次请求和一次请求的效果是一样的。如果用GET请求新建资源，可能会造成爬虫爬一次就新建一个资源的后果。所以发送表单如果正在loading, 刷新一下的时候浏览器会提醒你会重复发送表单数据。
3. **把数据裸露在URL中使GET更加不安全**。这里的安全是相对的，反正抓个包数据都能看见，因为浏览器会记录URL历史，一般网络服务商也只记录URL，在刚才的登陆例子中，如果是用GET方法发送登陆请求，那么只要你在别人电脑里登陆了一下，别人哪怕没什么技术能力，也能轻松看到你的密码。
4. **GET请求能被缓存，POST请求不能被缓存**。
5. **GET只允许 ASCII 字符，POST没有限制。也允许二进制数据**。

### 3.3 本质区别
V2EX上看到一个大哥说，GET和POST的区别就是，一个的方法是'GET'，一个的方法是'POST'。我觉得这个答案很有哲理性..很有佛教性空的味道。    

就是说：在http这个应用层协议上来说，两种被定义为不同的方法，但是从TCP这个传输层上说，他们并没有什么区别。给GET加上request body，给POST带上url参数？完全可以。

## 参考资料
+ [stackoverflow:http-get-with-request-body](https://stackoverflow.com/questions/978061/http-get-with-request-body)
+ [v2ex: 当你问到 GET 和 POST 的区别的时候，想听到什么样的答案？](https://www.v2ex.com/t/390315)
+ [知乎: 99%的人都理解错了HTTP中GET与POST的区别](https://zhuanlan.zhihu.com/p/22536382)
+ [blog:RESTful 架构风格概述](http://blog.igevin.info/posts/restful-architecture-in-general/)
+ [blog:RESTful API 编写指南](http://blog.igevin.info/posts/restful-api-get-started-to-write/)
+ [blog: best-practices-for-a-pragmatic-restful-api](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
+ [w3school](https://www.w3schools.com/tags/ref_httpmethods.asp)
+ [github:GET请求和POST请求的区别](https://github.com/zwhu/blog/issues/12)
+ [rfc2616](https://tools.ietf.org/html/rfc2616#section-9.3)
+ [stackoverflow: post-with-url-query good or not ](https://stackoverflow.com/questions/611906/http-post-with-url-query-parameters-good-idea-or-not)
