---
{"title":"Mybatis-Plus 全局Update更新策略，和insert插入策略","url":"https://www.modb.pro/db/126905","clipped_at":"2022-10-17 18:05:12","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/Mybatis-Plus-全局Update更新策略-和insert插入策略_1666001112/","dgPassFrontmatter":true}
---


# Mybatis-Plus 全局Update更新策略，和insert插入策略

# 前言

最近在使用`mybatis-plus`  
做项目的时候，发现使用`updatById`  
方法的时候，更新某个字段时候出现了问题，一般业务操作都是更新不为空的字段，结果发现更新了所有字段，这是由于mybatis-plus全局的更新策略导致的，我们可以通过相应全局配置来解决

![](/img/user/阅读库藏/assets/1666001112-40ed6fe791d25f010610e24ada9b84e2.png)看官方文档可知，数据库全局配置策略有三种，分别是查询策略，更新策略，和添加策略

点击这里进入官方文档

# 全局数据库策略配置

1.  配置
    

`#全局策略   mybatis-plus.global-config.db-config.update-strategy=not_empty   mybatis-plus.global-config.db-config.insert-strategy=not_empty   mybatis-plus.global-config.db-config.select-strategy=not_empty   `  

可选的配置值，看源码如下

`package com.baomidou.mybatisplus.annotation;      public enum FieldStrategy {       IGNORED,       NOT_NULL,       NOT_EMPTY,       DEFAULT,       NEVER;          private FieldStrategy() {       }   }   `  

> 1.  IGNORED 忽略判断，所有字段都进行更新和插入
>     
> 2.  NOT\_NULL只更新和插入非NULL值
>     
> 3.  NOT\_EMPTY 只更新和插入非NULL值且非空字符串
>     
> 4.  NEVER 永远不进行更新和插入
>     
> 5.  DEFAULT 默认NOT\_NULL
>     

默认取值，看源码可知

`public static class DbConfig {           private IdType idType;           private String tablePrefix;           private String schema;           private String columnFormat;           private String propertyFormat;           private boolean tableUnderline;           private boolean capitalMode;           private IKeyGenerator keyGenerator;           private String logicDeleteField;           private String logicDeleteValue;           private String logicNotDeleteValue;           private FieldStrategy insertStrategy;           private FieldStrategy updateStrategy;           private FieldStrategy selectStrategy;              public DbConfig() {               this.idType = IdType.ASSIGN_ID;               this.tableUnderline = true;               this.capitalMode = false;               this.logicDeleteValue = "1";               this.logicNotDeleteValue = "0";               this.insertStrategy = FieldStrategy.NOT_NULL;               this.updateStrategy = FieldStrategy.NOT_NULL;               this.selectStrategy = FieldStrategy.NOT_NULL;           }   `  

默认取值配置都是NOT\_NULL

## 更新策略配置

也就是我们在使用`updateById()`  
方法时候，在没有指定更新策略时候使用默认策略，为`NOT_NULL`  
，

也就是说当对象字段是`NULL`  
的时候不会进行set更新，如果我们字段是空字符串就会进行set更新操作，

所以我们可以更改我们全局配置不为空`not_empty`  
时候才更新

`mybatis-plus.global-config.db-config.update-strategy=not_empty   `  

也可以在需要的字段中单独指定字段更新策略

`/**        * 用户类型        */       @TableField(value = "ADMIN_TYPE_ID",updateStrategy = FieldStrategy.NOT_EMPTY)       private String userType;   `  

或者可以使用`UpdateWrapper`  
方式替换`updateById`  

![](/img/user/阅读库藏/assets/1666001112-43e6f43de0ab71279a32e8639a9738b8.png)

## 添加策略

同理我们在进行`inser`  
或者`save`  
，方法时候，在没有指定更新策略时候使用默认策略，为`NOT_NULL`  
，

也就是说当对象字段是`NULL`  
的时候不会进行ins添加值，如果我们字段是空字符串就会进行添加值操作，

我们也可以指定其他策略进行添加操作

  

[数据库](https://www.modb.pro/tag/%E6%95%B0%E6%8D%AE%E5%BA%93?type=knowledge)

文章转载自猿小叔，如果涉嫌侵权，请发送邮件至：contact@modb.pro进行举报，并提供相关证据，一经查实，墨天轮将立刻删除相关内容。

### 评论

### 相关阅读

[

国内数据库风云人物百人团，认识四十个以上就算你牛！

墨天轮编辑部

7724次阅读

2022-09-23 16:49:55



](https://www.modb.pro/db/498041)[

Hive SQL中常见的show语法有哪些？

Ingemar

926次阅读

2022-09-26 17:00:50



](https://www.modb.pro/db/506714)[

openGauss学习笔记- - -初识与使用技巧

叶秋

450次阅读

2022-09-22 12:39:35



](https://www.modb.pro/db/497394)[

软硬兼施，创新融合 | 2022年9月《中国数据库行业分析报告》精彩概览！

墨天轮编辑部

261次阅读

2022-09-22 12:09:38



](https://www.modb.pro/db/497379)[

瀚高股份与俄罗斯工程院丁治明院士共建院士工作站隆重签约

瀚高数据库

191次阅读

2022-09-25 14:24:20



](https://www.modb.pro/db/500107)[

华为认证考试 HCIE-GaussDB-OLTP V1.0 发布！

GaussDB数据库

176次阅读

2022-10-02 22:55:35



](https://www.modb.pro/db/507719)[

【我与openGauss的故事】基于openGauss的五子棋AI项目

winter

172次阅读

2022-10-07 10:54:12



](https://www.modb.pro/db/507973)[

解放军总医院：升级HIS数据库架构，提升数据库效率

通讯员

130次阅读

2022-09-21 18:30:51



](https://www.modb.pro/db/497256)[

细分领域创新引领，云和恩墨被正式认定为国家级专精特新“小巨人”企业

云和恩墨

113次阅读

2022-09-29 14:53:54



](https://www.modb.pro/db/505875)[

中核核信自主研发核信实时数据库，助力核工业企业数字化转型

核电那些事

104次阅读

2022-09-29 17:48:06



](https://www.modb.pro/db/506820)

[

猿

](https://www.modb.pro/u/311653)

[猿小叔](https://www.modb.pro/u/311653)

关注

[

40

文章

](https://www.modb.pro/u/311653)[

1

粉丝

](https://www.modb.pro/u/311653)[

8K+

浏览量

](https://www.modb.pro/u/311653)

热门文章

[

SpringBoot 默认json解析器详解和字段序列化自定义

2021-08-05 730浏览

](https://www.modb.pro/db/115624)[

SpringBoot之SpringSecurity权限注解在方法上进行权限认证多种方式

2021-09-28 621浏览

](https://www.modb.pro/db/126904)[

Java MD5和SHA256等常用加密算法

2021-09-29 462浏览

](https://www.modb.pro/db/126903)[

SpringBoot @ModelAttribute 用法

2021-08-09 435浏览

](https://www.modb.pro/db/115623)[

MySQL查询之内连接，外连接查询场景的区别与不同

2021-09-17 248浏览

](https://www.modb.pro/db/110889)

最新文章

[

Java MD5和SHA256等常用加密算法

2021-09-29 462浏览

](https://www.modb.pro/db/126903)[

SpringBoot之SpringSecurity权限注解在方法上进行权限认证多种方式

2021-09-28 621浏览

](https://www.modb.pro/db/126904)[

mybatis if else if 条件判断SQL片段表达式取值和拼接

2021-09-26 148浏览

](https://www.modb.pro/db/115609)[

mybatis的mapper特殊字符转移以及动态SQL条件查询

2021-09-24 241浏览

](https://www.modb.pro/db/115610)[

MySQL查询之内连接，外连接查询场景的区别与不同

2021-09-17 248浏览

](https://www.modb.pro/db/110889)