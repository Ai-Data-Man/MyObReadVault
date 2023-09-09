---
{"title":"BouncyCastle - 廖雪峰的官方网站","url":"https://www.liaoxuefeng.com/wiki/1252599548343744/1305362418368545","clipped_at":"2022-09-27 14:54:30","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/BouncyCastle-廖雪峰的官方网站_1664261670/","dgPassFrontmatter":true}
---


# BouncyCastle - 廖雪峰的官方网站

#### BouncyCastle

最后更新: 2022/6/16 10:24 / 阅读: 1531643

* * *

我们知道，Java标准库提供了一系列常用的哈希算法。

但如果我们要用的某种算法，Java标准库没有提供怎么办？

方法一：自己写一个，难度很大；

方法二：找一个现成的第三方库，直接使用。

[BouncyCastle](https://www.bouncycastle.org/)就是一个提供了很多哈希算法和加密算法的第三方库。它提供了Java标准库没有的一些算法，例如，RipeMD160哈希算法。

我们来看一下如何使用BouncyCastle这个第三方提供的算法。

首先，我们必须把BouncyCastle提供的jar包放到classpath中。这个jar包就是`bcprov-jdk18on-xxx.jar`，可以从[官方网站](https://www.bouncycastle.org/latest_releases.html)下载。

Java标准库的`java.security`包提供了一种标准机制，允许第三方提供商无缝接入。我们要使用BouncyCastle提供的RipeMD160算法，需要先把BouncyCastle注册一下：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 注册BouncyCastle:
        Security.addProvider(new BouncyCastleProvider());
        // 按名称正常调用:
        MessageDigest md = MessageDigest.getInstance("RipeMD160");
        md.update("HelloWorld".getBytes("UTF-8"));
        byte[] result = md.digest();
        System.out.println(new BigInteger(1, result).toString(16));
    }
}
```

其中，注册BouncyCastle是通过下面的语句实现的：

```javascript
Security.addProvider(new BouncyCastleProvider());
```

注册只需要在启动时进行一次，后续就可以使用BouncyCastle提供的所有哈希算法和加密算法。

### 练习

从[![](/img/user/阅读库藏/assets/1664261670-82df66c05e8151386ba91ee9702386ff.png)](https://gitee.com/)下载练习：[使用BouncyCastle提供的RipeMD160](https://gitee.com/liaoxuefeng/learn-java/blob/master/practices/Java%E6%95%99%E7%A8%8B/120.%E5%8A%A0%E5%AF%86%E4%B8%8E%E5%AE%89%E5%85%A8.1255943717668160/30.BouncyCastle.1305362418368545/encrypt-bc.zip?utm_source=blog_lxf) （推荐使用[IDE练习插件](https://www.liaoxuefeng.com/wiki/1252599548343744/1266092093733664)快速下载）

### 小结

BouncyCastle是一个开源的第三方算法提供商；

BouncyCastle提供了很多Java标准库没有提供的哈希算法和加密算法；

使用第三方算法前需要通过`Security.addProvider()`注册。

### 读后有收获可以支付宝请作者喝咖啡：

![](/img/user/阅读库藏/assets/1664261670-82635d7207da45f243d250a146511415.png)

### 还可以分享给朋友：

[分享到微博](#0)

* * *

[上一页](https://www.liaoxuefeng.com/wiki/1252599548343744/1304227729113121)[下一页](https://www.liaoxuefeng.com/wiki/1252599548343744/1305366354722849)

* * *