---
{"title":"bootstrap.yaml和application.yaml的区别","url":"http://www.manongjc.com/detail/25-sdhpuxlqpngxydd.html","clipped_at":"2022-11-28 18:45:04","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/bootstrap-yaml和application-yaml的区别_1669632304/","dgPassFrontmatter":true}
---

# bootstrap.yaml和application.yaml的区别

时间:2021-08-24

本文章向大家介绍bootstrap.yaml和application.yaml的区别，主要包括bootstrap.yaml和application.yaml的区别使用实例、应用技巧、基本知识点总结和需要注意事项，具有一定的参考价值，需要的朋友可以参考一下。

## bootstrap.yaml

配置一些引导系统启动的参数，这些参数一旦指定后就不会变动了。比如程序的端口号，配置中心的地址等。

## application.yaml

应用级别的参数配置，可能会根据业务需求做动态配置。比如日志级别，一些开关参数等。

## 加载的顺序

加入我们使用到配置中的话，我们还会涉及到很多配置文件。那么这些配置文件的加载顺序是怎么样的呢？

这里我做了个实验，使用nacos做配置中心，一共涉及到下面几个配置文件：

nacos配置中心的相关配置如下：

```ruby
spring:
  profiles:
    active: @profiles.active@
  application:
    name: payment-service-dubbo-nacos
  main:
    allow-bean-definition-overriding: true
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
        file-extension: yaml
        # 配置注册在tpag这个用户专有的namespace
        namespace: 6f97a206-ce19-44c2-85be-c601170d306e
        group: ${spring.application.name}
        username: tpag
        password: tpag
        refresh-enabled: true
        # 这边的shared-config和ext-config不能配置namespace，和上面的配置共享一个namespace，所以一般用于读取一个团队内部的共享文件
        extensionConfigs[0]:
          data-id: ext1.yaml
          refresh: true
          # 默认是DEFAULT_GROUP
          group: ${spring.application.name}
        extensionConfigs[1]:
          data-id: ext2.yaml
          refresh: true
          group: ${spring.application.name}
        shared-configs[0]:
          data-id: share1.yaml
          refresh: true
          group: ${spring.application.name}
        shared-configs[1]:
          data-id: share2.yaml
          refresh: true
          group: ${spring.application.name}
```

这几个配置文件加载的顺序是 bootstrap.yml > application.yml > application-dev.yml > share1.yaml > share2.yaml > ext1.yaml > ext2.yaml > cloud:nacos:config 标签下面dataId指定的配置文件。

假如配置文件中有相同的配置，后加载的配置会覆盖先加载的配置，所以如果使用Nacos配置中心的话，nacos上的配置的优先级会比较高。

人生的主旋律其实是苦难，快乐才是稀缺资源。在困难中寻找快乐，才显得珍贵~

原文地址：https://www.cnblogs.com/54chensongxia/p/15180894.html