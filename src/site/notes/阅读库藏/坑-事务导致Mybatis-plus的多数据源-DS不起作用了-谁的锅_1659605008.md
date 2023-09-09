---
{"title":"坑，事务导致Mybatis plus的多数据源@DS不起作用了，谁的锅","url":"https://baijiahao.baidu.com/s?id=1685490934094994913&wfr=spider&for=pc","clipped_at":"2022-08-04 17:23:28","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/坑-事务导致Mybatis-plus的多数据源-DS不起作用了-谁的锅_1659605008/","dgPassFrontmatter":true}
---


# 坑，事务导致Mybatis plus的多数据源@DS不起作用了，谁的锅

坑，事务导致Mybatis plus的多数据源@DS不起作用了，谁的锅

播报文章

[![](/img/user/阅读库藏/assets/1659605008-188d280939d40aa34a804074c62f47f6.jpg)](https://author.baidu.com/home?from=bjh_article&app_id=1685480472361044)

[

zmyHow

](https://author.baidu.com/home?from=bjh_article&app_id=1685480472361044)

2020-12-08 15:02

关注

![](/img/user/阅读库藏/assets/1659605008-eac6987a965111a051a71d50fbd60bf8.jpeg)

由于使用了微服务，会有多个数据库的情况，有时业务需要，需要切换数据源，所以使用了Mybatis plus的@DS来切换多数据源

yml数据库配置如下：

![](/img/user/阅读库藏/assets/1659605008-10d2936951455f761f05f8f0ad9817fd.jpeg)

service如下，默认是master数据源

![](/img/user/阅读库藏/assets/1659605008-72cc32dc5f774ccdf6424ebfd0183979.png)

userService

![](/img/user/阅读库藏/assets/1659605008-b4e8503506542d9304f9ad6e7094f456.png)

bookService

![](/img/user/阅读库藏/assets/1659605008-445177b5e95ab8e496561574c68fc4e1.png)

但是神奇的事发生的，bookService的数据库应该是common，但是却是master的，也就是说@DS切换数据源没有起作用

于是开始排查

去除MasterService.upload上面的@Transactional，数据源切换正常，但是事务无效BookService的save上面加@Transactional，数据源没有切换BookService的save上面加@Transactional(propagation = Propagation.REQUIRES\_NEW)，数据源切换，且事务有效原因：

开启事务的同时，会从数据库连接池获取数据库连接；如果内层的service使用@DS切换数据源，只是又做了一层拦截，但是并没有改变整个事务的连接在这个事务内的所有数据库操作，都是在事务连接建立之后，所以会产生数据源没有切换的问题为了使@DS起作用，必须替换数据库连接，也就是改变事务的传播机智，产生新的事务，获取新的数据库连接所以bookService的save方法上除了加@Transactional外，还需要设置propagation = Propagation.REQUIRES\_NEW使得代码走以下逻辑：

![](/img/user/阅读库藏/assets/1659605008-f6be9d0bed7353b29e422d44c9219456.png)

在走startTransaction，再走doBegin，重新创建新事务，获取新的数据库连接，从而得到@DS的数据源

![](/img/user/阅读库藏/assets/1659605008-f00eba24fa28bb8fd1c741925fc8058c.png)

![](/img/user/阅读库藏/assets/1659605008-86c6a2bf2dd4d9df895547909a1e5726.png)

最终代码如下，只需要修改的是bookService

在方法上增加：@Transactional(propagation = Propagation.REQUIRES\_NEW)

![](/img/user/阅读库藏/assets/1659605008-69415628595f1cd0b1a7b92dd0de40ef.png)

@DS数据源切换生效@Transaction事务生效需要注意：

master：userService

common：bookService

common数据库的操作，需要在master之后，这样当bookService.save失败，会使得userService回滚； 如果common的操作先，那当userService失败，无法使bookService回滚

会回滚

@Transactional(rollbackFor = Exception.class)

public void upload(ReqDto respDto){

userService.save(respDto);

bookService.save(respDto);

}

不会回滚

![](/img/user/阅读库藏/assets/1659605008-a30930d42693b6a50d4d4e9afeb03166.png)

举报/反馈

## 发表评论

发表

## 评论列表（5条）

##### vancepaulzhang

事务注解和DS切换数据源组合成一个注解，方便使用，例如：@Target({ElementType.TYPE, ElementType.METHOD})@Retention(RetentionPolicy.RUNTIME)@Documented@Transactional(readOnly = true, propagation = Propagation.REQUIRES\_NEW)@DS(ConfigConstants.ACCOUNT\_DS\_NAME)public @interface AccountRead {}

2021-06-11

回复

1

##### tangjun147

我用你的成功了，凡是出现@Transactional的地方，就再单独加个@DS，其他不变

05-25 17:49

江苏

回复

赞

##### loveqin79

我也遇到这个问题，感谢分享，节约我很多时间

2021-04-21

回复

4

##### songhr86115

@DS支持配置bean吗

2021-02-01

回复

赞

##### 百度网友2ee8a01

感谢

2021-06-21

回复

赞

没有更多啦

## 作者最新文章

[![](/img/user/阅读库藏/assets/1659605008-0ce60f66cabe45734b8bd317b5de3500.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8776426826313307093%22%7D&n_type=1&p_from=3)

[

### 再也不用自己封装工具类，这个神级工具类太香了

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8776426826313307093%22%7D&n_type=1&p_from=3)

2021-12-0417阅读

[![](/img/user/阅读库藏/assets/1659605008-13a7c63354f3db41be6b700f8b98742c.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9903704107866225694%22%7D&n_type=1&p_from=3)

[

### 面试官问：什么是浅拷贝和深拷贝？

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9903704107866225694%22%7D&n_type=1&p_from=3)

2021-10-0455阅读

[![](/img/user/阅读库藏/assets/1659605008-0179b997f3ffaba48d3412f5d9beea98.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9410246506266149137%22%7D&n_type=1&p_from=3)

[

### git提交也有规范，看下大神是怎么做的

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9410246506266149137%22%7D&n_type=1&p_from=3)

2021-09-19163阅读

## 相关推荐

[![](/img/user/阅读库藏/assets/1659605008-7ccea62388d37aedc7351c6b4d0409c6.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9058363335946393247%22%7D&n_type=1&p_from=4)

[

### Java位运算符详解

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9058363335946393247%22%7D&n_type=1&p_from=4)

[李兴华java2python](https://author.baidu.com/home?from=bjh_article&app_id=1713931719838145)[](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9058363335946393247%22%7D&n_type=1&p_from=4)

[![](/img/user/阅读库藏/assets/1659605008-510ba61f06d55cfbc7e2fecb68ee90f4.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9855667174987597886%22%7D&n_type=1&p_from=4)

[

### 单例模式（Singleton）

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9855667174987597886%22%7D&n_type=1&p_from=4)

[司波达也Owen](https://author.baidu.com/home?from=bjh_article&app_id=1691189041795568)[](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9855667174987597886%22%7D&n_type=1&p_from=4)

[![](/img/user/阅读库藏/assets/1659605008-f3ccc78dcbe234ed0a6d384022d4f30d.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8956929329439863448%22%7D&n_type=1&p_from=4)

[

### 带你学习Python编程开发工具

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8956929329439863448%22%7D&n_type=1&p_from=4)

[脚本之家](https://author.baidu.com/home?from=bjh_article&app_id=1549322409310619)[](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8956929329439863448%22%7D&n_type=1&p_from=4)

[![](/img/user/阅读库藏/assets/1659605008-aaaa8b26af8ceaf08d8ad9cfbc52cfdf.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8790721048648411885%22%7D&n_type=1&p_from=4)

[

### 微软 Win10 七月可选更新导致无法更换输入法，已进行回滚修复

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8790721048648411885%22%7D&n_type=1&p_from=4)

[IT之家](https://author.baidu.com/home?from=bjh_article&app_id=1551599273641720)[](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8790721048648411885%22%7D&n_type=1&p_from=4)

[![](/img/user/阅读库藏/assets/1659605008-85fe07b4410369767f850978c4e8fd17.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8773390997295745188%22%7D&n_type=1&p_from=4)

[

### termius破解，windows和mac版本，2022.08.03最新详细教程+安装包

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8773390997295745188%22%7D&n_type=1&p_from=4)

[安哥说前端](https://author.baidu.com/home?from=bjh_article&app_id=1715424594647401)[](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8773390997295745188%22%7D&n_type=1&p_from=4)

[](https://top.baidu.com/board?platform=pc&sa=pcindex_entry)

换一换

*   1[东部战区实弹训练原声现场画面](https://www.baidu.com/s?wd=%E4%B8%9C%E9%83%A8%E6%88%98%E5%8C%BA%E5%AE%9E%E5%BC%B9%E8%AE%AD%E7%BB%83%E5%8E%9F%E5%A3%B0%E7%8E%B0%E5%9C%BA%E7%94%BB%E9%9D%A2&sa=fyb_news_feedpc&rsv_dl=fyb_news_feedpc&from=feedpc)爆

*   2[台媒：解放军军演增至7处 时间延长](https://www.baidu.com/s?wd=%E5%8F%B0%E5%AA%92%EF%BC%9A%E8%A7%A3%E6%94%BE%E5%86%9B%E5%86%9B%E6%BC%94%E5%A2%9E%E8%87%B37%E5%A4%84+%E6%97%B6%E9%97%B4%E5%BB%B6%E9%95%BF&sa=fyb_news_feedpc&rsv_dl=fyb_news_feedpc&from=feedpc)沸

*   3[强国必须强军，军强才能国安](https://www.baidu.com/s?wd=%E5%BC%BA%E5%9B%BD%E5%BF%85%E9%A1%BB%E5%BC%BA%E5%86%9B%EF%BC%8C%E5%86%9B%E5%BC%BA%E6%89%8D%E8%83%BD%E5%9B%BD%E5%AE%89&sa=fyb_news_feedpc&rsv_dl=fyb_news_feedpc&from=feedpc)

*   4[解放军重要军事演训行动已开始](https://www.baidu.com/s?wd=%E8%A7%A3%E6%94%BE%E5%86%9B%E9%87%8D%E8%A6%81%E5%86%9B%E4%BA%8B%E6%BC%94%E8%AE%AD%E8%A1%8C%E5%8A%A8%E5%B7%B2%E5%BC%80%E5%A7%8B&sa=fyb_news_feedpc&rsv_dl=fyb_news_feedpc&from=feedpc)热

*   5[外交部：不再安排中日外长金边会晤](https://www.baidu.com/s?wd=%E5%A4%96%E4%BA%A4%E9%83%A8%EF%BC%9A%E4%B8%8D%E5%86%8D%E5%AE%89%E6%8E%92%E4%B8%AD%E6%97%A5%E5%A4%96%E9%95%BF%E9%87%91%E8%BE%B9%E4%BC%9A%E6%99%A4&sa=fyb_news_feedpc&rsv_dl=fyb_news_feedpc&from=feedpc)

*   6[东部战区导弹全部精准命中目标](https://www.baidu.com/s?wd=%E4%B8%9C%E9%83%A8%E6%88%98%E5%8C%BA%E5%AF%BC%E5%BC%B9%E5%85%A8%E9%83%A8%E7%B2%BE%E5%87%86%E5%91%BD%E4%B8%AD%E7%9B%AE%E6%A0%87&sa=fyb_news_feedpc&rsv_dl=fyb_news_feedpc&from=feedpc)

*   7[解放军台海演习航母、核潜艇参演](https://www.baidu.com/s?wd=%E8%A7%A3%E6%94%BE%E5%86%9B%E5%8F%B0%E6%B5%B7%E6%BC%94%E4%B9%A0%E8%88%AA%E6%AF%8D%E3%80%81%E6%A0%B8%E6%BD%9C%E8%89%87%E5%8F%82%E6%BC%94&sa=fyb_news_feedpc&rsv_dl=fyb_news_feedpc&from=feedpc)

*   8[泽连斯基称想与中国“直接对话”](https://www.baidu.com/s?wd=%E6%B3%BD%E8%BF%9E%E6%96%AF%E5%9F%BA%E7%A7%B0%E6%83%B3%E4%B8%8E%E4%B8%AD%E5%9B%BD%E2%80%9C%E7%9B%B4%E6%8E%A5%E5%AF%B9%E8%AF%9D%E2%80%9D&sa=fyb_news_feedpc&rsv_dl=fyb_news_feedpc&from=feedpc)

*   9[被问为台带来何意义？佩洛西沉默](https://www.baidu.com/s?wd=%E8%A2%AB%E9%97%AE%E4%B8%BA%E5%8F%B0%E5%B8%A6%E6%9D%A5%E4%BD%95%E6%84%8F%E4%B9%89%EF%BC%9F%E4%BD%A9%E6%B4%9B%E8%A5%BF%E6%B2%89%E9%BB%98&sa=fyb_news_feedpc&rsv_dl=fyb_news_feedpc&from=feedpc)

*   10[华春莹说G7的列强梦该醒醒了](https://www.baidu.com/s?wd=%E5%8D%8E%E6%98%A5%E8%8E%B9%E8%AF%B4G7%E7%9A%84%E5%88%97%E5%BC%BA%E6%A2%A6%E8%AF%A5%E9%86%92%E9%86%92%E4%BA%86&sa=fyb_news_feedpc&rsv_dl=fyb_news_feedpc&from=feedpc)

## 相关推荐

[![](/img/user/阅读库藏/assets/1659605008-7ccea62388d37aedc7351c6b4d0409c6.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9058363335946393247%22%7D&n_type=1&p_from=4)

[

### Java位运算符详解

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9058363335946393247%22%7D&n_type=1&p_from=4)

[李兴华java2python](https://author.baidu.com/home?from=bjh_article&app_id=1713931719838145)[](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9058363335946393247%22%7D&n_type=1&p_from=4)

[![](/img/user/阅读库藏/assets/1659605008-510ba61f06d55cfbc7e2fecb68ee90f4.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9855667174987597886%22%7D&n_type=1&p_from=4)

[

### 单例模式（Singleton）

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9855667174987597886%22%7D&n_type=1&p_from=4)

[司波达也Owen](https://author.baidu.com/home?from=bjh_article&app_id=1691189041795568)[](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_9855667174987597886%22%7D&n_type=1&p_from=4)

[![](/img/user/阅读库藏/assets/1659605008-f3ccc78dcbe234ed0a6d384022d4f30d.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8956929329439863448%22%7D&n_type=1&p_from=4)

[

### 带你学习Python编程开发工具

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8956929329439863448%22%7D&n_type=1&p_from=4)

[脚本之家](https://author.baidu.com/home?from=bjh_article&app_id=1549322409310619)[](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8956929329439863448%22%7D&n_type=1&p_from=4)

[![](/img/user/阅读库藏/assets/1659605008-aaaa8b26af8ceaf08d8ad9cfbc52cfdf.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8790721048648411885%22%7D&n_type=1&p_from=4)

[

### 微软 Win10 七月可选更新导致无法更换输入法，已进行回滚修复

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8790721048648411885%22%7D&n_type=1&p_from=4)

[IT之家](https://author.baidu.com/home?from=bjh_article&app_id=1551599273641720)[](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8790721048648411885%22%7D&n_type=1&p_from=4)

[![](/img/user/阅读库藏/assets/1659605008-85fe07b4410369767f850978c4e8fd17.jpg)](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8773390997295745188%22%7D&n_type=1&p_from=4)

[

### termius破解，windows和mac版本，2022.08.03最新详细教程+安装包

](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8773390997295745188%22%7D&n_type=1&p_from=4)

[安哥说前端](https://author.baidu.com/home?from=bjh_article&app_id=1715424594647401)[](https://mbd.baidu.com/newspage/data/landingsuper?context=%7B%22nid%22%3A%22news_8773390997295745188%22%7D&n_type=1&p_from=4)