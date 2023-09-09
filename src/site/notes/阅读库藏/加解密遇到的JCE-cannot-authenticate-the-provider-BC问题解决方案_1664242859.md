---
{"title":"加解密遇到的JCE cannot authenticate the provider BC问题解决方案","url":"https://blog.csdn.net/nathan_csdn/article/details/116262224","clipped_at":"2022-09-27 09:40:59","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/加解密遇到的JCE-cannot-authenticate-the-provider-BC问题解决方案_1664242859/","dgPassFrontmatter":true}
---


# 加解密遇到的JCE cannot authenticate the provider BC问题解决方案

# 前言

> 相信搞过加解密的同学大部分都会遇到过这个问题——JCE cannot authenticate the provider BC  
> 笔者最近在做一个亚马逊的项目需要进行GPG加解密，因为测试jar包是亚马逊提供的，jar是经过签名的，咱也不能修改，所以只能修改自己的JDK配置

# 分析

在解密时错误日志如下：

```bash
Exception in thread "main" java.lang.IllegalArgumentException: Could not decrypt the provided file, please check that provided keys are correct and in the format as specified in tech spec.Exception decrypting key
	at com.amazon.psp.datavalidation.file.FileUtils.getDataAsStream(FileUtils.java:37)
	at com.amazon.psp.datavalidation.entrypoint.MainClass.main(MainClass.java:22)
Caused by: org.bouncycastle.openpgp.PGPException: Exception decrypting key
	at org.bouncycastle.openpgp.PGPSecretKey.extractKeyData(Unknown Source)
	at org.bouncycastle.openpgp.PGPSecretKey.extractPrivateKey(Unknown Source)
	at org.bouncycastle.openpgp.PGPSecretKey.extractPrivateKey(Unknown Source)
	at org.bouncycastle.openpgp.PGPSecretKey.extractPrivateKey(Unknown Source)
	at com.amazon.psp.datavalidation.pgp.PgpDecryptionHelper.decryptAndVerifySignature(PgpDecryptionHelper.java:68)
	at com.amazon.psp.datavalidation.file.FileUtils.getDataAsStream(FileUtils.java:26)
	... 1 more
Caused by: java.lang.SecurityException: JCE cannot authenticate the provider BC
	at javax.crypto.Cipher.getInstance(Cipher.java:656)
	at org.bouncycastle.jcajce.ProviderJcaJceHelper.createCipher(Unknown Source)
	at org.bouncycastle.openpgp.operator.jcajce.OperatorHelper.createCipher(Unknown Source)
	at org.bouncycastle.openpgp.operator.jcajce.JcePBESecretKeyDecryptorBuilder$1.recoverKeyData(Unknown Source)
	... 7 more
Caused by: java.util.jar.JarException: file:/Users/xxxxx/Desktop/case/PSPPTestPackage-1.8.jar has unsigned entries - ATTRIBUTIONS.txt
	at javax.crypto.JarVerifier.verifySingleJar(JarVerifier.java:502)
	at javax.crypto.JarVerifier.verifyJars(JarVerifier.java:363)
	at javax.crypto.JarVerifier.verify(JarVerifier.java:289)
	at javax.crypto.JceSecurity.verifyProviderJar(JceSecurity.java:164)
	at javax.crypto.JceSecurity.getVerificationResult(JceSecurity.java:190)
	at javax.crypto.Cipher.getInstance(Cipher.java:652)
	... 10 more
```

看到这个不用慌，因为只是缺少一个jar而已，**添加一个jar，修改一下JDK安全配置**就搞定。

这里提醒一句：**如果修改后不生效，记得多换几个版本！！！**

大家可以看到`bouncycastle`包抛出的异常，我们可以去maven仓库[https://mvnrepository.com/search?q=bouncycastle](https://mvnrepository.com/search?q=bouncycastle)下载一下这个jar，搜索后会出现如下页面。  
![在这里插入图片描述](/img/user/阅读库藏/assets/1664242859-bb05fbacc3a9dceec9902a5621797a52.png)  
maven配置如下，因为我是通过maven来下载这个jar的，你要是有其他渠道可以拿到jar就可以不用我这种方式。

```xml
    <dependency>
      <groupId>org.bouncycastle</groupId>
      <artifactId>bcprov-jdk15on</artifactId>
      <version>1.50</version>
    </dependency> 
```

我们选择第3个，至于为什么选择它，稍后再说。

## 修改JDK配置

找到自己本地JDK安装根目录，进入`jre/lib/security`目录，编辑`java.security`文件; 操作如下：

```bash
# 因为我是Linux和MacOS记得要加 sudo ，windows下直接编辑内容即可

# 进入JDK目录
cd /Library/Java/JavaVirtualMachines/jdk1.8.0_211.jdk/Contents/Home/jre/lib/security

# 编辑security文件
sudo vim java.security
```

打开文件后，添加如下配置：

```bash
# 这个序号11根据自己的配置写就行，有可能你的配置和我不一样
security.provider.11=org.bouncycastle.jce.provider.BouncyCastleProvider
```

人家本身是有其他安全配置的，添加完咱们的配置后，配置长这样：

```bash
# There must be at least one provider specification in java.security.
# There is a default provider that comes standard with the JDK. It
# is called the "SUN" provider, and its Provider subclass
# named Sun appears in the sun.security.provider package. Thus, the
# "SUN" provider is registered via the following:
#
#    security.provider.1=sun.security.provider.Sun
#
# (The number 1 is used for the default provider.)
#
# Note: Providers can be dynamically registered instead by calls to
# either the addProvider or insertProviderAt method in the Security
# class.

#
# List of providers and their preference orders (see above):
#
security.provider.1=sun.security.provider.Sun
security.provider.2=sun.security.rsa.SunRsaSign
security.provider.3=sun.security.ec.SunEC
security.provider.4=com.sun.net.ssl.internal.ssl.Provider
security.provider.5=com.sun.crypto.provider.SunJCE
security.provider.6=sun.security.jgss.SunProvider
security.provider.7=com.sun.security.sasl.Provider
security.provider.8=org.jcp.xml.dsig.internal.dom.XMLDSigRI
security.provider.9=sun.security.smartcardio.SunPCSC
security.provider.10=apple.security.AppleProvider
security.provider.11=org.bouncycastle.jce.provider.BouncyCastleProvider
#
# Sun Provider SecureRandom seed source.
```

截图如下：  
![在这里插入图片描述](/img/user/阅读库藏/assets/1664242859-debc8589a1ddf4ad994e62b854638306.png)

## 添加依赖

将刚才下载好的jar文件（ **bcprov-jdk15on-1.50.jar** ），复制到`jre/lib/ext/`目录下  
![在这里插入图片描述](/img/user/阅读库藏/assets/1664242859-25cc55ece7a8eb01ec206cdf89f59034.png)  
然后就OK了！

至于刚才说为啥知道下载这个jar，是因为亚马逊的解密的代码用的版本是`1.50`，而只有`org.bouncycastle » bcprov-jdk15on`跟我的版本一致！

> 另外扩展一个思路：如果你不知道第三方提供的解密工具类是哪个版本，尝试去反编译，看下它的源码，看看打包出的jar是哪年的？  
> 然后对应maven仓库里的依赖找到对应的年份。