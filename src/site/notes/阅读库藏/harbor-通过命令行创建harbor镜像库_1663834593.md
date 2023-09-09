---
{"title":"harbor 通过命令行创建harbor镜像库","url":"https://www.cnblogs.com/xiaoli-ya/p/16172008.html","clipped_at":"2022-09-22 16:16:33","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/harbor-通过命令行创建harbor镜像库_1663834593/","dgPassFrontmatter":true}
---

# [harbor 通过命令行创建harbor镜像库](https://www.cnblogs.com/xiaoli-ya/p/16172008.html)

> 网上查了几个，发现都是写的同样的，并且不能用。'

## 通过命令行创建仓库 （亲测有效）

```powershell
# 创建k8s仓库
curl -k -u 'admin:Harbor12345' -XPOST -H "Content-Type:application/json" -d '{"project_name":"k8s"}' "https://Harbor-IP:Port/api/v2.0/projects"
```

## 未验证

常用操作

```r
\l                        #列出所有数据库
\c dbname      #切换数据库
\d                      #列出当前数据库的所有表
\q                      #退出数据库
```

1.首先进入到harbor的db库里

docker exec -it harbor-db bash

2.执行psql -w 输入数据库的密码，这步直接回车就可以了

> 默认密码是“root123”

3.执行\\l 查询 所有库

4.执行\\c 加库名  进入到该库

5.进入到数据库里执行\\d 列出库里所有的表

6.select \* from 表名   可以查看到相关表内的所有数据,一般查询project这张表里可以查看到项目是否创建成功了么

下面执行最关键的一步，创建harbor库的项目，创建一个kube\_system的项目名，以此类推只需修改id和项目名就可以依次创建harbor库的项目了

```sql
insert into project(project_id,owner_id,name) values('2','1','kube_system');
insert into project_metadata(id,project_id,name,value) values('2','2','public','true');
insert into project_member(id,project_id,entity_id,entity_type,role) values('2','2','1','u','1');
insert into quota(id,reference,reference_id,hard) values('2','project','2','{"storage": -1}');
insert into quota_usage(id,reference,reference_id,used) values('2','project','2','{"storage": 0}');      
```

创建google\_containers项目

```sql
insert into project(project_id,owner_id,name) values('3','1','google_containers');
insert into project_metadata(id,project_id,name,value) values('3','3','public','true');
insert into project_member(id,project_id,entity_id,entity_type,role) values('3','3','1','u','1');
insert into quota(id,reference,reference_id,hard) values('3','project','3','{"storage": -1}');
insert into quota_usage(id,reference,reference_id,used) values('3','project','3','{"storage": 0}');
```

[好文要顶](javascript:) [关注我](javascript:) [收藏该文](javascript:)

[![](/img/user/阅读库藏/assets/1663834593-4fa19b59ab0e49fad43d0f8d508d5d7c.png)](https://home.cnblogs.com/u/xiaoli-ya/)

[小李y](https://home.cnblogs.com/u/xiaoli-ya/)  
[粉丝 - 0](https://home.cnblogs.com/u/xiaoli-ya/followers/) [关注 - 2](https://home.cnblogs.com/u/xiaoli-ya/followees/)  

[+加关注](javascript:)

0

0

[«](https://www.cnblogs.com/xiaoli-ya/p/16144791.html) 上一篇： [根证书、服务器证书、用户证书的区别](https://www.cnblogs.com/xiaoli-ya/p/16144791.html "发布于 2022-04-14 15:22")  
[»](https://www.cnblogs.com/xiaoli-ya/p/16300405.html) 下一篇： [mysqldump报错information\_schema (1109)](https://www.cnblogs.com/xiaoli-ya/p/16300405.html "发布于 2022-05-23 10:55")

posted @ 2022-04-20 22:10    阅读(101)  评论(0)     

[刷新评论](javascript:)[刷新页面](#)[返回顶部](#top)

登录后才能查看或发表评论，立即 [登录](javascript:) 或者 [逛逛](https://www.cnblogs.com/) 博客园首页

**编辑推荐：**  
· [单标签实现复杂的棋盘布局](https://www.cnblogs.com/coco1s/p/16710203.html)  
· [C# 非托管泄漏中 HEAP\_ENTRY 的 Size 对不上是怎么回事？](https://www.cnblogs.com/huangxincheng/p/16707620.html)  
· [CSS之垂直水平居中的背后](https://www.cnblogs.com/zaking/p/16532299.html)  
· [记一次 .NET 某打印服务 非托管内存泄漏分析](https://www.cnblogs.com/huangxincheng/p/16691715.html)  
· [使用 Three.js 实现一个创意纪念页面](https://www.cnblogs.com/dragonir/p/16691845.html)  

**最新新闻**：  
· [索尼预热 PlayStation VR2，新机将于明年初发布](https://news.cnblogs.com/n/728750/)  
· [羊了个羊，无法穿过的狭沟](https://news.cnblogs.com/n/728749/)  
· [在Web3，大厂是要被鄙视的](https://news.cnblogs.com/n/728748/)  
· [小马智行、曹操出行、吉利智驾中心达成战略合作，联合打造Robotaxi车队](https://news.cnblogs.com/n/728747/)  
· [普通人用了上瘾，艺术家看了流泪：这个爆火的AI真能一键挑战大师画作？](https://news.cnblogs.com/n/728746/)  
» [更多新闻...](https://news.cnblogs.com/ "IT 新闻")