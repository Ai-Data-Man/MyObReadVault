---
{"title":"java stream distinct() 按指定对象属性进行去重","url":"https://www.cnblogs.com/unknows/p/13534953.html","clipped_at":"2022-08-23 15:22:15","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/java-stream-distinct-按指定对象属性进行去重_1661239335/","dgPassFrontmatter":true}
---

*   [博客园](https://www.cnblogs.com/)
*   [首页](https://www.cnblogs.com/unknows/)
*   [新随笔](https://i.cnblogs.com/EditPosts.aspx?opt=1)
*   [联系](https://msg.cnblogs.com/send/%E5%B0%B1%E8%BF%99%E4%B8%AA%E5%90%8D%E5%AD%97%E5%A5%BD)
*   [管理](https://i.cnblogs.com/)
*   [订阅](javascript:) [![订阅](/img/user/阅读库藏/assets/1661239335-cc83ec48276e68764e387655ed57cf3a.gif)](https://www.cnblogs.com/unknows/rss/)

随笔- 125  文章- 0  评论- 19  阅读- 64万 

# [java stream distinct() 按指定对象属性进行去重](https://www.cnblogs.com/unknows/p/13534953.html)

**方式一**

1\. distinct（）不提供按照属性对对象列表进行去重的直接实现。它是基于hashCode（）和equals（）工作的。如果我们想要按照对象的属性，对对象列表进行去重，我们可以通过其它方法来实现

```java
public static <T> Predicate<T> distinctByKey(Function<? super T, ?> keyExtractor) {
        Map<Object,Boolean> seen = new ConcurrentHashMap<>();
        return t -> seen.putIfAbsent(keyExtractor.apply(t), Boolean.TRUE) == null;
} 
```

2\. 使用方法：用Stream接口的 filter()接收为参数

```plain
List<Entity> lists = list.stream().filter(distinctByKey(b->b.getTid())).collect(Collectors.toList());
```

**方式二**

存在重复数据的问题，这里使用stream流的衍生功能，去除一个对象中的部分元素的重复如下：

```plain
ArrayList<ProductProcessDrawbackDto> collect = records1.stream().collect(Collectors.collectingAndThen(
                        Collectors.toCollection(() -> new TreeSet<>(
                                Comparator.comparing(
                                        ProductProcessDrawbackDto::getId))), ArrayList::new));
```

其中records1是处理的对象，改对象的list集合，collect是处理后返回的结果

其中的ProductProcessDrawbackDto是处理的list中每一个对象，id是判断是否重复的条件（去除id相同的重复元素，只保留一条）

多个字段或者多个条件去重

```plain
 ArrayList<PatentDto> collect1 = patentDtoList.stream().collect(Collectors.collectingAndThen(
                Collectors.toCollection(() -> new TreeSet<>(
                        Comparator.comparing(p->p.getPatentName() + ";" + p.getLevel()))), ArrayList::new)
```

参考：[https://blog.csdn.net/yojofly/article/details/100986216](https://blog.csdn.net/yojofly/article/details/100986216)

[http://www.10qianwan.com/articledetail/605000.html](http://www.10qianwan.com/articledetail/605000.html)

分类: [Java](https://www.cnblogs.com/unknows/category/1053623.html)

[好文要顶](javascript:) [关注我](javascript:) [收藏该文](javascript:) [![](/img/user/阅读库藏/assets/1661239335-3212f7b914cc9773fb30bbf4656405fc.png)](javascript: "分享至新浪微博") [![](/img/user/阅读库藏/assets/1661239335-cb7153d1c13a5d9aef10ebab342f6f71.png)](javascript: "分享至微信")

[![](/img/user/阅读库藏/assets/1661239335-7e60824a7def941a8c2a502a6c7a88ae.png)](https://home.cnblogs.com/u/unknows/)

[就这个名字好](https://home.cnblogs.com/u/unknows/)  
[粉丝 - 66](https://home.cnblogs.com/u/unknows/followers/) [关注 - 19](https://home.cnblogs.com/u/unknows/followees/)  

[+加关注](javascript:)

5

0

[«](https://www.cnblogs.com/unknows/p/13371294.html) 上一篇： [fastjson对象，JSON，字符串，map之间的互转](https://www.cnblogs.com/unknows/p/13371294.html "发布于 2020-07-24 12:18")  
[»](https://www.cnblogs.com/unknows/p/13638515.html) 下一篇： [mybatis中各种数据对象的映射关系](https://www.cnblogs.com/unknows/p/13638515.html "发布于 2020-09-09 14:21")

posted @ 2020-08-20 14:41  [就这个名字好](https://www.cnblogs.com/unknows/)  阅读(24221)  评论(1)  [编辑](https://i.cnblogs.com/EditPosts.aspx?postid=13534953)  [收藏](javascript:)  [举报](javascript:)

[刷新评论](javascript:)[刷新页面](#)[返回顶部](#top)

登录后才能查看或发表评论，立即 [登录](javascript:) 或者 [逛逛](https://www.cnblogs.com/) 博客园首页

**编辑推荐：**  
· [ASP.NET Core 6 框架揭秘实例演示：异常处理高阶用法](https://www.cnblogs.com/artech/p/inside-asp-net-core-6-33x.html)  
· [彻底了解线程池的原理——40行从零开始自己写线程池](https://www.cnblogs.com/Chang-LeHung/p/16599776.html)  
· [\[MAUI\]在窗口（页面）关闭后获取其返回值](https://www.cnblogs.com/lesliexin/p/16588060.html)  
· [.net 温故知新：IOC控制反转，DI依赖注入](https://www.cnblogs.com/SunSpring/p/16601339.html)  
· [架构与思维：互联网高性能 Web 架构](https://www.cnblogs.com/wzh2010/p/15730040.html)  

**最新新闻**：  
· [周期理论看互联网](https://news.cnblogs.com/n/727010/)  
· [小红书8年梦碎电商：有命种草，没命变现](https://news.cnblogs.com/n/727000/)  
· [回乡下种地的年轻人：治愈了，也迷茫了](https://news.cnblogs.com/n/727003/)  
· [国产AI作画神器火了，更懂中文，竟然还能做周边！](https://news.cnblogs.com/n/726997/)  
· [2022手机半年考，vivo夺冠，荣耀起飞](https://news.cnblogs.com/n/727008/)  
» [更多新闻...](https://news.cnblogs.com/ "IT 新闻")