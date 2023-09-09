---
{"title":"mysql支不支持fulljoin_mysql full join 报错（无效）解决方式","url":"https://blog.csdn.net/weixin_32376927/article/details/114860757","clipped_at":"2022-05-28 10:17:37","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/mysql支不支持fulljoin_mysql-full-join-报错-无效-解决方式_1653704257/","dgPassFrontmatter":true}
---


# mysql支不支持fulljoin_mysql full join 报错（无效）解决方式

前些天在做数据统计的时想使用全表链接查询，于是使用full join 链接但是报错了，而其他的left join和right join是正常的，这样可以肯定的是full join出了问题。难道是mysql不支持full join？查看mysql手册，答案是mysql 不支持full in。

![29e82ff8c46bc34097412cae910a45de.png](/img/user/阅读库藏/assets/1653704257-9cda214230fe898ae581405d32597690.png)

在网上一些资料会介绍full join的用法，但是那是在特定的数据库中才能使用的sql关键词，(例如，oracal支持full in)。

那么有什么方式实现full in想要得到的结果呢？

使用left join 和 right join。full in 可以相当与左右链接的结果。

具体实现代码(示例)

select \* from t1 left join t2 on t1.id = t2.id

union

select \* from t1 right join t2 on t1.id = t2.id

左右链接合并查询得到结果和full in预期的结果是一致的，当然full in的测试结果只能借助其他支持它的数据库啦。

出现这些问题其实质是sql中的某些关键词不兼容造成的，mysql手册也指出尽量使用left join去替换right join，以保持代码的跨数据库移植，说明left join的通用性比right join强。这些同时也在提醒我们在网上查看资料是要分析是否适用你当下的环境。