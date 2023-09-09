---
{"title":"SpringCloud(34)——Nacos分组配置","url":"https://www.jianshu.com/p/dc474882cd0b","clipped_at":"2022-07-06 16:59:53","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/SpringCloud-34-——Nacos分组配置_1657097993/","dgPassFrontmatter":true}
---


# SpringCloud(34)——Nacos分组配置

### 1.问题描述

①实际开发中，通常一个系统会准备多个环境  
dev开发环境  
test测试环境  
prod生产环境  
那么如何保证指定环境启动时服务能正确读取到Nacos上响应环境的配置文件呢？  
②一个大型分布式微服务系统会有很多微服务子项目，  
每个微服务项目又会有相应的多个环境  
那么怎么对这些微服务配置进行管理呢？

### 2.NameSpace、Group和DataID

![](/img/user/阅读库藏/assets/1657097993-7b01a4a0f5fd6f5226be79b4d44ce952.png)

image.png

我们通过控制台可以看到Nacos里面有NameSpace、Group和DataID这几个属性  
**那么他们之间有什么关系呢？**  
首先，这三个属性类似于Java里面的package名和类名  
最外层的namespace是可以用于区分部署环境的，Group和DataID逻辑上区分两个目标对象。  

![](/img/user/阅读库藏/assets/1657097993-4707a155fcad3eeb48fa5d9abc408534.png)

image.png

**默认情况**  
Namespace=public，Group=DEFAULT\_GROUP，Cluster是DEFAULT

Nacos默认的命名空间是public，Namespace主要用来实现隔离。  
比方说我们现在有三个环境：开发、测试、生产环境，我们就可以创建三个Namespace，不同的Namespace之间是隔离的。

Group默认是DEFAULT\_GROUP，Group可以把不同的微服务划分到同一分组里面去。

Service就是微服务；一个Service可以包含多个Cluster，Nacos默认Cluster是DEFAULT，是Cluster是对指定微服务的一个虚拟划分。

### 3.dataID层的配置

我们在Nacos的配置管理里面再新建一个配置

![](/img/user/阅读库藏/assets/1657097993-4ca248b45f7462a883e76b8bf00746b0.png)

image.png

![](/img/user/阅读库藏/assets/1657097993-b561a9319d68ff58ee170cc5cb9cd68a.png)

image.png

然后我们可以看到在默认的Group和Namespace下面现在有两个配置  
一个是dev的，一个是test的。  
现在修改服务里面的配置

![](/img/user/阅读库藏/assets/1657097993-d26ec313b7ce7b5979ab5cf8f5b2ce81.png)

image.png

我们要使用test的配置，在启动的时候active改成test就可以了，我们重启再测试一下

![](/img/user/阅读库藏/assets/1657097993-ab38e942a5bc08fe48b22e73616aa90e.png)

image.png

我们可以看到test的配置文件已经读取到了。

### 4.group层的配置

首先还是要在nacos里新建一个配置

![](/img/user/阅读库藏/assets/1657097993-87841d69f5543b91514ef6f678620c02.png)

image.png

新建的配置Group要叫DEV，要和原来的DEFAULT\_GROUP区别开。

接着。代码里面要配置使用新的配置文件

![](/img/user/阅读库藏/assets/1657097993-fdfd6bbb9a5b5a3ac0b3e55a95f0ec2f.png)

image.png

在bootstrap中配置一个group参数即可

![](/img/user/阅读库藏/assets/1657097993-b37ec19ae0bc6ce5df37058be244fb0f.png)

image.png

在application里面要配置在nacos里创建的配置文件的后缀名  
接下来我们启动测试了。

![](/img/user/阅读库藏/assets/1657097993-06ad8832531a871fa525bfdb6b4e4bcf.png)

image.png

访问同样的接口，可以看到返回的参数是不同的

### 5.Namespace层

##### 5.1nacos配置

对于namespace层，我们需要先创建命名空间

![](/img/user/阅读库藏/assets/1657097993-1eb7f726907f3d737a77da0ddc613903.png)

image.png

![](/img/user/阅读库藏/assets/1657097993-56e03e8f26712253b57e643e3a5faea0.png)

image.png

记住我们的命名空间ID，后面我们在代码里配置的时候要用到  
命名空间建好了之后我们可以看到

![](/img/user/阅读库藏/assets/1657097993-04f0043e2a206fb8e4788b7335d6c9e8.png)

image.png

配置列表上面多了一个我们新建的namespace  
我们在这个命名空间下再新建一个配置

![](/img/user/阅读库藏/assets/1657097993-8d6928251926e5a708767bc728992253.png)

image.png

##### 5.2工程代码配置

![](/img/user/阅读库藏/assets/1657097993-9dd627695fbae2fa11a11d178febc57b.png)

image.png

在bootstrap里面配置我们刚刚创建的group和namespaceID

![](/img/user/阅读库藏/assets/1657097993-c61775eb9090ad794cfe825b96e9aa8e.png)

image.png

在application里面同样要改成和新建配置的后缀名一样的test。

##### 5.3测试结果

![](/img/user/阅读库藏/assets/1657097993-78a318c721ed35bedfeb5d936016c390.png)

image.png

返回的值就是我们新建的命名空间中的配置！