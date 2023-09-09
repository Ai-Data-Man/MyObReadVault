---
{"title":"SpringCloud使用Nacos作为配置中心如何使本地配置优先","url":"https://blog.csdn.net/xhaimail/article/details/122084999","clipped_at":"2022-10-11 10:11:07","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/SpringCloud使用Nacos作为配置中心如何使本地配置优先_1665454267/","dgPassFrontmatter":true}
---


# SpringCloud使用Nacos作为配置中心如何使本地配置优先

# **分享知识 传递快乐**

在项目中使用了 SpringCloud 配置中心模式时远程配置的[优先级](https://so.csdn.net/so/search?q=%E4%BC%98%E5%85%88%E7%BA%A7&spm=1001.2101.3001.7020)默认高于本地配置，如果想要通过本地配置改变远程配置**一定**要在**远程配置**中做一下配置：

**以 nacos 为例：**

```bash
spring:
  cloud:
    config:
      # 如果本地配置优先级高，那么 override-none 设置为 true，包括系统环境变量、本地配置文件等配置
      override-none: true
      # 如果想要远程配置优先级高，那么 allow-override 设置为 false，如果想要本地配置优先级高那么 allow-override 设置为 true
      allow-override: true
      # 只有系统环境变量或者系统属性才能覆盖远程配置文件的配置，本地配置文件中配置优先级低于远程配置；注意本地配置文件不是系统属性
      override-system-properties: false
```

**注意：**一定要配置到远程配置中，否则不生效

**优先级如下：**

1.  命令行参数
2.  java:comp/env 里的 JNDI 属性
3.  JVM 系统属性
4.  系统环境变量
5.  RandomValuePropertySource 属性类生成的 random.\* 属性
6.  应用以外的 application.properties（或 yml）文件
7.  打包在应用内的 application.properties（或 yml）文件
8.  在应用 @Configuration 配置类中，用 @PropertySource 注解声明的属性文件
9.  SpringApplication.setDefaultProperties 声明的默认属性

—————————  
如有不足请留言指正  
相互学习，共同进步