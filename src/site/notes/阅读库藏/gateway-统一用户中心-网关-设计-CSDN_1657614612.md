---
{"title":"gateway 统一用户中心 网关 设计 - CSDN","url":"https://www.csdn.net/tags/OtDakg4sMjk0MzktYmxvZwO0O0OO0O0O.html","clipped_at":"2022-07-12 16:30:12","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/gateway-统一用户中心-网关-设计-CSDN_1657614612/","dgPassFrontmatter":true}
---


# gateway 统一用户中心 网关 设计 - CSDN

# 一、简介

  **Spring Cloud Gateway 使用的Webflux中的reactor-netty响应式编程组件，底层使用了Netty通讯框架**  
  Gateway是在Spring生态系统之上构建的API网关服务,于Spring 5, Spring Boot 2和Project Reactor等技术。SpringCloud Gateway作为Spring Cloud生态系统中的网关,目标是替代Zuul,在Spring Cloud 2.0以上版本中,没有对新版本的Zuul 2.0以上最新高性能版本进行集成，仍然还是使用的Zuul 1.x非Reactor模式的老版本。而为了提升网关的性能，SpringCloud Gateway是**基于WebFlux框架实现的**，而WebFlux框架底层则使用了高性能的**Reactor模式通信框架Netty**。在1 .x版本中都是采用的Zuul网关;但在2.x版本中，zuul的升级一直跳票， SpringCloud最后自己研发了一个网关替代Zuul,那就是SpringCloud Gateway一句话: **Gateway是zuul1.x版的替代。**  
  Gateway旨在提供一种简单而有效的方式来对API进行路由， 以及**提供一些强大的过滤器功能**， 例如:熔断、限流、重试等。

![在这里插入图片描述](/img/user/阅读库藏/assets/1657614612-98e99c42f00e4b77372a9f978b3c7058.png)

## 1.1 网关在微服务架构中的位置：

![在这里插入图片描述](/img/user/阅读库藏/assets/1657614612-109aa15c1785c5e7b93801d34f57cf16.png)

## 1.2 API Gateway 的作用

  APIGateway 即API网关，所有请求首先会经过这个网关，然后到达后端服务，有点类似于Facade模式。**API网关作为系统接口对外的统一出口，可以减少调用方对服务实现的感知。**  
  在没有API网关作为统一出口的情况下，需要调用方自己组合各种服务，而且容易让调用方感知后端各种服务的存在。在加入了API网关之后，通过网关暴露接口给调用方，**调用方可以在不感知后端服务的情况下调用服务，而且通过统一的接口**，后端服务接口的变化不会影响调用方，后端服务变化可以通过网关的转换，对外仍然保持一致的风格。

### 1.2.1 统一对外接口

  当用户需要集成不同产品或者服务之间的功能，调用不同服务提供的能力。利用APIGateway可以**让用户在不感知服务边缘的情况下，利用统一的接口组装服务**。 对于公司内部不同的服务，提供的接口可能在风格上存在一定的差异，通过APIGateway可以统一这种差异。 当内部服务修改时，可以通过APIGateway进行适配，不需要调用方进行调整 **减少对外暴露服务可以增加系统安全性。**

### 1.2.2 统一鉴权

  通过APIGateway**对访问进行统一鉴权**，不需要每个应用单独对调用方进行鉴权，应用可以专注业务。

### 1.2.3 服务注册与授权

  可以控制调用方可以使用和不可以使用的服务。

### 1.2.4 服务限流

  通过APIGateway可以对调用方调用每个接口的每日调用及总调用次数限制。

### 1.2.5 全链路跟踪

  通过APIGateway提供的唯一请求Id，监控调用流程，以及调用的响应时间。  
![在这里插入图片描述](/img/user/阅读库藏/assets/1657614612-02c8bfe16c0999b9244bd299bf39bb56.png)

# 二、开源API网关框架

## 2.1 spring zuul

   Zuul是Netflix出品的一个基于JVM路由和服务端的负载均衡器。它的主要功能有：认证、压力测试、金丝雀测试、动态路由、负载削减、安全、静态响应处理和主动交换管理。spring zuul 是spring Cloud的组件，可以和spring cloud的各个组件结合使用。  
   在1 .x版本中都是采用的Zuul网关;但在2.x版本中，zuul的升级一直跳票，于是就有了spring cloud gateway。  
   Spring Cloud中所集成的Zuul版本，采用的是Tomcat容器,使用的是**传统的Servlet I0处理模型。**  
**Servlet的生命周期**：servlet由servlet container进行生命周期管理。  
container启动时构造servlet对象并调用servlet init0进行初始化;  
container运行时接受请求，并为每个请求分配一 个线程(一般从线程池中获取空闲线程)然后调用service(）。  
container关闭时调用servlet destory0销毁servlet;  
![在这里插入图片描述](/img/user/阅读库藏/assets/1657614612-d5577403040871a70980a75cc61c3ade.png)  
**上述模式的缺点:**  
   servlet堤一个简单的网络I0模型, 当请求进入servlet container时, servlet container就会为其绑定一个线程， 在并发不高的场景下这种模型是适用的。但是一旦高并发(比如抽风,用jemeter压测),线程数量就会，上涨，而线程资源代价是昂贵的(上线文切换,内存消耗大)严重影响请求的处理时间。在一些简单业务场景下,不希望为每个request分配一个线程, 只需要1个或几个线程就能应对极大并发的请求,这种业务场景下servlet模型没有优势所以Zuul 1.X是基于servlet之上的一个**阻塞式处理模型**，即spring实现了处理所有request请求的一个servlet (DispatcherServlet) 并由该servlet阻塞式处理处理。所以Springcloud Zuul无法摆脱servlet模型的弊端

## 2.2 spring cloud gateway

  Gateway是基于**异步非阻塞模型**上进行开发的,性能方面不需要担心。虽然Netflix早就发布了最新的Zuul 2.x,但Spring Cloud貌似没有整合计划。而且Netflix相关组件都宣布进入维护期。  
**Spring Cloud Gateway具有如下特性: .**

*   基于Spring Framework 5, Project Relactor和Spring Boot 2.0 进行构建;
*   动态路由:能够匹配任何请求属性;
*   可以对路由指定Predicate (断言)和Filter (过滤器) ;
*   集成Hystrix的断路器功能;
*   集成Spring Cloud服务发现功能;
*   易于编写的Predicate (断言)和Filter (过滤器) ;
*   请求限流功能; .
*   支持路径重写。

### WebFlex:

  传统的Web框架，比如说: struts2, springmvc等 都是基于Servlet API与Servlet容器基础之上运行的。但是在Servlet3.1之后有了异步非阻塞的支持。而**WebFlux是一个 典型非阻塞异步的框架**,它的核心是基于Reactor的相关API实现的。相对于传统的web框架来说，它可以运行在诸如Netty, Undertow及 支持Servlet3.1的容器上。非阻塞式+函数式编程(Spring5必须让你使用java8)  
  Spring WebFlux是Spring 5.0引入的**新的响应式框架**,区别于Spring MVC,它**不需要依赖Servlet API,它是完全异步非阻塞的，并且基于Reactor来实现响应式流规范。**

## 2.3 Spring Cloud Gateway与Zuul的区别

  在SpringCloud Finchley正式版之前，Spring Cloud推荐的网关是Netflix提供的Zuul。

*   Zuul1.x, 是一个基于阻塞 I/O 的API Gateway
*   Zuul 1.x基于Servlet 2. 5使用阻塞架构它不支持任何长连接(如WebSocket) Zuul的设计模式和Nginx较像，每次 I/0 操作都是从工作线程中选择一个执行， 请求线程被阻塞到工作线程完成，但是差别是NginxC++实现, Zuul 用Java实现，而JVM本身会有第一次加载较慢的情况，使得Zuul的性能相对较差。
*   Zuul 2.x理念更先进,想基于Netty非阻塞和支持长连接，但SpringCloud目前还没有整合。 Zuul 2.x的性能较Zuul 1.x有较大提升。在性能方面,根据官方提供的基准测试， Spring Cloud Gateway的RPS (每秒请求数)是Zuul的1.6倍。
*   Spring Cloud Gateway建立在Spring Framework 5、Project Reactor和Spring Boot2之上，使用非阻塞API。
*   Spring Cloud Gateway还持WebSocket，组与Spring紧密集成拥有 更好的开发体验

# 三、Spring Cloud Gateway 三大核心概念

## 3.1 Route(路由)

  路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由。

## 3.2 Predicate（断言）

  参考的是java8的java.util.function.Predicate开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由。

## 3.3 Filter(过滤)

  指的是Spring框架中GatewayFilter的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

## 总体

![在这里插入图片描述](/img/user/阅读库藏/assets/1657614612-100529501756476595ebc759d36ce665.png)  
  web请求通过一些匹配条件,定位到真正的服务节点。并在这个转发过程的前后,进行一些精细化控制。**predicate就是我们的匹配条件**；而**filter 就可以理解为一个无所不能的拦截器**。有了这两个元素,再加上目标url ,就可以实现一个具体的路由了。

# 四、Gateway工作流程

## 4.1 官网介绍：

![在这里插入图片描述](/img/user/阅读库藏/assets/1657614612-78064fcdb3358fa245061db547a134cf.png)

*   客户端向Spring Cloud Gateway发出请求。然后在Gateway Handler Mapping中找到与请求相匹配的路由,将其发送到Gateway Web Handler。
*   Handler再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。
*   过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前( “pre” )或之后( “post” )执行业务逻辑。**Filter在"pre" 类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转换等,在"post"类型的过滤器中可以做响应内容、响应头的修改，日志的输出,流量监控等**有着非常重要的作用。

## 4.2 Gateway 网关路由（Route）两种配置方式

### 4.2.1 通过配置文件配置\*\*

静态绑定配置，**静态路由**：

```java
server:
  port: 9527
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      routes:
    - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
      uri: http://localhost:8001   #匹配后提供服务的路由地址
      predicates:
        - Path=/payment/get/**   #断言,路径相匹配的进行路由

    - id: payment_routh2
      uri: http://localhost:8001
      predicates:
        - Path=/payment/lb/**   #断言,路径相匹配的进行路由

eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

通过微服务名实现**动态路由**：  
  默认情况下Gateway会根据注册中心的服务列表，以**注册中心上微服务名为路径创建动态路由进行转发**，从而实现动态路由的功能。  
  需要注意的是**uri的协议为lb，表示启用Gateway的负载均衡功能。** lb://serviceName是spring cloud gateway在微服务中自动为我们创建的负载均衡uri。

```java
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001   #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/get/**   #断言,路径相匹配的进行路由
        - id: payment_routh2
          #uri: http://localhost:8001   #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/lb/**   #断言,路径相匹配的进行路由
eureka:
  instance:
    hostname: cloud-gateway-service
  client:
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

### 4.2.2 通过Java代码中注入RouteLocator的Bean

```java
@Configuration
public class GateWayConfig {
    /**
     * 配置了一个id为route -name的路由规则,
     * 当访问地址http://localhost:9527/guonei时会自动转发到地址: http://news.baidu.com/guonei
     * @param routeLocatorBuilder
     * @return
     */
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder) {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        routes.route("path_rote_atguigu", r -> r.path("/guonei").uri("http://news.baidu.com/guonei")).build();
        return routes.build();
    }

    @Bean
    public RouteLocator customRouteLocator2(RouteLocatorBuilder routeLocatorBuilder) {
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        routes.route("path_rote_atguigu2", r -> r.path("/guoji").uri("http://news.baidu.com/guoji")).build();
        return routes.build();
    }
```

## 4.3 断言（Predicate）的使用

  Spring Cloud Gateway将路由匹配作为Spring WebFlux HandlerMapping基础架构的一部分。S**pring Cloud Gateway包括许多内置的Route Predicate工厂**。所有这些Predicate都与HTTP请求的不同属性匹配。多个RoutePredicate工厂可以进行组合。  
  Spring Cloud Gateway创建Route对象时，使用RoutePredicateFactory创建Predicate对象, Predicate 对象可以赋值给Route。Spring Cloud Gateway包含许多内置的Route Predicate Factories.所有这些谓词都匹配HTTP请求的不同属性。**多种谓词工厂可以组合，并通过逻辑and。**  
  通过这些断言的配置，就可以控制 http 请求哪些有效，及在什么条件下有效。说白了，**Predicate就是为了实现一组匹配规则，让请求过来找到对应的Route进行处理。**

**配置方式**如下格式：**增加 predicates 标签**，主要有以下几个属性断言

```java
server:
  port: 9527

spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true  #开启从注册中心动态创建路由的功能，利用微服务名进行路由
      routes:
        - id: payment_routh #路由的ID，没有固定规则但要求唯一，建议配合服务名
          #uri: http://localhost:8001   #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/get/**   #断言,路径相匹配的进行路由

        - id: payment_routh2
          #uri: http://localhost:8001   #匹配后提供服务的路由地址
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/lb/**   #断言,路径相匹配的进行路由
```

### 4.3.1 After Route Predicate

路由在\[设置时间\]之后有效

```java
- After=2020-07-18T10:59:34.102+08:00[Asia/Shanghai]  #路由什么时候起效
```

### 4.3.2 Before Route Predicate

路由在\[设置时间\]之前有效

```java
- Before=2020-07-28T10:59:34.102+08:00[Asia/Shanghai] #路由有效截止日期
```

### 4.3.3 Between Route Predicate

路由在\[设置时间\]之间有效

```java
- Between=2020-07-18T10:59:34.102+08:00[Asia/Shanghai],2020-07-28T10:59:34.102+08:00[Asia/Shanghai] #有效时间段
```

### 4.3.4 Cookie Route Predicate

  Cookie Route Predicate需要两个参数, 一个是Cookie name ,一个是正则表达式。路由规则会通过获取对应的Cookie name值和正则表达式去匹配，如果**匹配上就会执行路由，如果没有匹配上则不执行**

```java
- Cookie=username,zzyy  #请求信息携带Cookie信息与此匹配
```

### 4.3.5 Header Route Predicate

  两个参数: 一个是属性名称和一个正则表达式，这个属性值和正则表达式匹配则执行。

```java
- Header=X-Request-Id,\d+ #请求头要有X-Request- Id属性并且值为整数的正则表达式
```

### 4.3.6 Host Route Predicate

主机号相同才匹配

```java
- Host=**.eureka.com	#主机号相同才匹配
```

### 4.3.7 Method Route Predicate

请求方式相同则匹配

```java
- Method=GET #请求方式相同则匹配
```

### 4.3.8 Path Route Predicate

断言,路径相匹配的进行路由

```java
- Path=/payment/lb/**   #断言,路径相匹配的进行路由
```

### 4.3.9 Query Route Predicate

要有参数名称并且是正整数才能路由

```java
- Query=username, \d+ #要有参数名称并且是正整数才能路由
```

## 4.4 过滤（Filter）的使用

  路由过滤器可用于**修改进入的HTTP请求和返回的HTTP响应,路由过滤器只能指定路由进行使用。** Spring Cloud Gateway内置了多种路由过滤器,他们都由GatewayFilter的工厂 类来产生。

### 4.4.1 生命周期

只有两个，**pre：在业务逻辑之前执行；post：在业务逻辑之后执行**

### 4.4.2 单一过滤器----GatewayFilter

官网可知，有31种之多。  
![在这里插入图片描述](/img/user/阅读库藏/assets/1657614612-caff13c0d42d86b85d321fde17b9cf5d.png)  
配置方式：

```java
spring:
 application:
	name: cloud -gateway
cloud:
 gateway:
	locator:
		enabled: true #开启从注册中心动态创建路由的功能
		lower-case-service-id: true #使用小写服务名，默认是大写
	routes:
		- id: payment_ routh #payment_ route #路由的ID， 没有固定规则但要求唯- ，建议配合服务名
		uri: lb://cloud - provider-payment #匹配后的目标服务地址，供服务的路由地址
		#uri: http://localhost: 8001 #匹配后提供服务的路由地址
		filters:
			AddRequestParameter=X-Request-Id, 1024 #过滤器工厂会在匹配的请求头加上-对请求头，名称为X-Request- Id值为1024
		predicates:
			- Path=/ paymentInfo/** #断言，路径相匹配的进行路由
			- Method=GET, POST
```

### 4.4.3 全局过滤器----GlobalFilter

有十种，具体请看官网。  
![在这里插入图片描述](/img/user/阅读库藏/assets/1657614612-2b87871067e9424a960dc1874697cc8c.png)

### 4.4.4 自定义过滤器

**自定义全局GlobalFilter：**  
编写Java 类实现 GlobalFilter和Orderd 接口

```java
@Component
@Slf4j
public class MyLogGateWayFilter implements GlobalFilter,Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {

        log.info("*********come in MyLogGateWayFilter: "+new Date());
        String uname = exchange.getRequest().getQueryParams().getFirst("username");
        if(StringUtils.isEmpty(username)){
            log.info("*****用户名为Null 非法用户,(┬＿┬)");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);//给人家一个回应
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }
	/**
     * 过滤器加载的顺序 越小,优先级别越高 *
     * @return
     */
    @Override
    public int getOrder() {
        return 0;
    }
}
```

这样就可以在所有请求执行具体方法之前和之后做一些操作，如全局日志记录、统一网关鉴权等。和Servlet的Filter具有类似的功能。