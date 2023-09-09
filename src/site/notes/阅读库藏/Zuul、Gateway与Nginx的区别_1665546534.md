---
{"title":"Zuul、Gateway与Nginx的区别","url":"https://blog.csdn.net/weixin_45525272/article/details/125945303","clipped_at":"2022-10-12 11:48:54","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/Zuul、Gateway与Nginx的区别_1665546534/","dgPassFrontmatter":true}
---

# Zuul、Gateway与Nginx的区别

![](/img/user/阅读库藏/assets/1665546534-426136e8363e68fc321f3d3551b0d579.png)

[流楚丶格念](https://yangyongli.blog.csdn.net/) ![](/img/user/阅读库藏/assets/1665546534-829b50e1a754811a0f05afff88b5db50.png) 于 2022-07-23 11:29:28 发布 ![](/img/user/阅读库藏/assets/1665546534-12234b4519a7e1441526c49ab7fc9a0d.png) 300 ![](/img/user/阅读库藏/assets/1665546534-169ac251df55845562af7f2f9151a130.png) 收藏 2

分类专栏： [\# SpringCloud](https://blog.csdn.net/weixin_45525272/category_11913621.html) 文章标签： [gateway](https://so.csdn.net/so/search/s.do?q=gateway&t=blog&o=vip&s=&l=&f=&viparticle=) [nginx](https://so.csdn.net/so/search/s.do?q=nginx&t=blog&o=vip&s=&l=&f=&viparticle=) [java](https://so.csdn.net/so/search/s.do?q=java&t=blog&o=vip&s=&l=&f=&viparticle=)

版权

 [![](/img/user/阅读库藏/assets/1665546534-9dc0d4a9f5a5c9c7c5f4dad57879b216.png) SpringCloud 专栏收录该内容](https://blog.csdn.net/weixin_45525272/category_11913621.html "SpringCloud")

23 篇文章 1 订阅

订阅专栏

### 文章目录

*   [导言](#_2)
*   [Nginx 和 网关 的区别](#Nginx____8)
*   *   [相同点:](#_12)
    *   [不同点:](#_20)
*   [Zuul 与 Gateway](#Zuul__Gateway_37)
*   *   [Zuul：](#Zuul_38)
    *   [Gateway：](#Gateway_44)
    *   [相同点](#_49)
    *   [不同点](#_54)
    *   [总结](#_78)

# 导言

> **首先Zuul、Gateway是一类，都是web servlet网关，处理的是http请求**
> 
> **而Nginx属于服务器web网关，处理的也是http请求**

# [Nginx](https://so.csdn.net/so/search?q=Nginx&spm=1001.2101.3001.7020) 和 网关 的区别

> 下面的网关都用[Zuul](https://so.csdn.net/so/search?q=Zuul&spm=1001.2101.3001.7020)替代，Nginx与Gateway也是大同小异

## 相同点:

1.  可以实现负载均衡 (Zuul使用的是Ribbon实现负载均衡)
    
2.  可以实现反向代理 (即隐藏真实ip地址)
    
3.  可以过滤请求,实现网关的效果
    

## 不同点:

1.  首先 , Nginx是C语言开发,而 Zuul 是Java语言开发
    
2.  其次,Nginx负载均衡实现,采用服务器实现负载均衡,而Zuul负载均衡的实现是采用 Ribbon + Eureka 来实现本地负载均衡.
    
3.  Nginx适合于服务器端负载均衡,Zuul适合[微服务](https://so.csdn.net/so/search?q=%E5%BE%AE%E6%9C%8D%E5%8A%A1&spm=1001.2101.3001.7020)中实现网关
    
4.  Nginx相比Zuul功能会更加强大,因为Nginx整合一些脚本语言( Nginx + lua )
    
5.  Nginx 是一个高性能的HTTP 和反向代理服务器, 也是一个 IMAP / POP3 /SMIP 服务器.  
    Zuul是 Spring Cloud Netflix 中的开源的一个API Gateway 服务器,本质上是一个web servlet 应用, 提供动态路由,监控,弹性,安全等边缘服务的框架. Zuul 相当于是设备和Netflix 流应用的 Web 网站后端所有请求的前门
    

# Zuul 与 Gateway

## Zuul：

使用的是阻塞式的 API，不支持长连接，比如 websockets。  
底层是servlet，Zuul处理的是http请求  
没有提供异步支持，流控等均由hystrix支持。  
依赖包spring-cloud-starter-netflix-zuul。

## Gateway：

Spring Boot和Spring Webflux提供的Netty底层环境，不能和传统的Servlet容器一起使用，也不能打包成一个WAR包。  
依赖spring-boot-starter-webflux和/ spring-cloud-starter-gateway  
提供了异步支持，提供了抽象负载均衡，提供了抽象流控，并默认实现了RedisRateLimiter。

## 相同点

1、底层都是servlet

2、两者均是web网关，处理的是http请求

## 不同点

1、内部实现：

gateway对比zuul多依赖了spring-webflux，在spring的支持下，功能更强大，内部实现了限流、负载均衡等，扩展性也更强，但同时也限制了仅适合于Spring Cloud套件

zuul则可以扩展至其他微服务框架中，其内部没有实现限流、负载均衡等。

2、是否支持异步

zuul仅支持同步  
gateway支持异步。

> 理论上gateway则更适合于提高系统吞吐量（但不一定能有更好的性能），最终性能还需要通过严密的压测来决定

3、框架设计的角度

gateway具有更好的扩展性，并且其已经发布了2.0.0的RELESE版本，稳定性也是非常好的

4、性能

gateway中的 WebFlux 模块的名称是 spring-webflux，名称中的 Flux 来源于 Reactor 中的类 Flux。Spring webflux 有一个全新的非堵塞的函数式 Reactive Web 框架，可以用来构建异步的、非堵塞的、事件驱动的服务，在伸缩性方面表现非常好。使用非阻塞API。 Websockets得到支持，并且由于它与Spring紧密集成，所以将会是一个更好的 开发 体验。

Zuul 1.x，是一个基于阻塞io的API Gateway。Zuul已经发布了Zuul 2.x，基于Netty，也是非阻塞的，支持长连接，但Spring Cloud暂时还没有整合计划。

## 总结

总的来说，在微服务[架构](https://so.csdn.net/so/search?q=%E6%9E%B6%E6%9E%84&spm=1001.2101.3001.7020)，如果使用了Spring Cloud生态的基础组件，则Spring Cloud Gateway相比而言更加具备优势，单从流式编程+支持异步上就足以让开发者选择它了。

对于小型微服务架构或是复杂架构（不仅包括微服务应用还有其他非Spring Cloud服务节点），zuul也是一个不错的选择。

![](/img/user/阅读库藏/assets/1665546534-2ece5aa028250a4d87a1d5b80a1b9677.png)

君子以言有物而行有恒

![](/img/user/阅读库藏/assets/1665546534-f909ae66c3ed503e3eb545adada2ed7d.png) QQ名片

![](/img/user/阅读库藏/assets/1665546534-9f010b7e0345df31af9285ce67a5a83a.png)

  

  

显示推荐内容

[![](/img/user/阅读库藏/assets/1665546534-4b7bc6d16d19e2cbde8eaf453a45d833.png)流楚丶格念](https://yangyongli.blog.csdn.net/)

[关注](javascript:)

*    ![](/img/user/阅读库藏/assets/1665546534-7758a3093834d492e2d75950c9101032.png) 0
*   ![](/img/user/阅读库藏/assets/1665546534-d82ff9be37cc360bfbff695ee0855b91.png)
*    [![](/img/user/阅读库藏/assets/1665546534-6f06c54c05c98d49b6a6d81042a39fa5.png) 2](javascript:)
*   [![打赏](/img/user/阅读库藏/assets/1665546534-52dc5d356ab6d0b35434e3b26f6cc860.png)](javascript:)
*    [![](/img/user/阅读库藏/assets/1665546534-89aae22a687cc2b4164d861b31a9ebc8.png) 0](#commentBox)

*   [![](/img/user/阅读库藏/assets/1665546534-ad402df0a23d656db1ebe7c061aae928.png)](javascript:)

专栏目录