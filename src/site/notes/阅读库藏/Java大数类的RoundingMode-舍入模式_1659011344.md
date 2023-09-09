---
{"title":"Java大数类的RoundingMode（舍入模式）","url":"https://blog.csdn.net/bailu666666/article/details/79829902","clipped_at":"2022-07-28 20:29:04","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/Java大数类的RoundingMode-舍入模式_1659011344/","dgPassFrontmatter":true}
---


# Java大数类的RoundingMode（舍入模式）

**Java大数类的RoundingMode（舍入模式）**  

  
        java.math.RoundingMode：这是一种枚举类型，它定义了8种数据的舍入模式。它与java.math.BigDecimal类中定义的8个同名静态常量的作用相同，可用BigDecimal.setScale(int newScale, RoundingMode roundingMode)来设置数据的精度和舍入模式。  
        

**1、ROUND\_UP：向远离零的方向舍入。**

        若舍入位为**非零**，则对舍入部分的前一位数字加1；若舍入位为**零**，则直接舍弃。即为**向外取整模式**。

**2、ROUND\_DOWN：向接近零的方向舍入。**

        不论舍入位是否为零，都直接舍弃。即为**向内取整模式。**

**3、ROUND\_CEILING：向正无穷大的方向舍入。**

        若 [BigDecimal](https://so.csdn.net/so/search?q=BigDecimal&spm=1001.2101.3001.7020) 为正，则舍入行为与 ROUND\_UP 相同；若为负，则舍入行为与 ROUND\_DOWN 相同。即为**向上取整模式**。

**4、ROUND\_FLOOR：向负无穷大的方向舍入。**

        若 BigDecimal 为正，则舍入行为与 ROUND\_DOWN 相同；若为负，则舍入行为与 ROUND\_UP 相同。即为**向下取整模式**。

**5、ROUND\_HALF\_UP：向“最接近的”整数舍入。**

        若舍入位**大于等于5**，则对舍入部分的前一位数字加1；若舍入位**小于5**，则直接舍弃。即为**四舍五入模式**。

**6、ROUND\_HALF\_DOWN**：向“最接近的”**整数**舍入。****

        若舍入位**大于5**，则对舍入部分的前一位数字加1；若舍入位**小于等于5**，则直接舍弃。即为**五舍六入模式**。

**7、ROUND\_HALF\_EVEN**：向“最接近的”**整数**舍入。****

        若（舍入位**大于5）**或者（舍入位**等于5**且前一位为**奇数**），则对舍入部分的前一位数字加1；

        若（舍入位**小于5）**或者（舍入位**等于5**且前一位为**偶数**），则直接舍弃。即为**银行家舍入模式**。

**8、ROUND\_UNNECESSARY**

        断言请求的操作具有精确的结果，因此不需要舍入。

        如果对获得精确结果的操作指定此舍入模式，则抛出ArithmeticException。

### **不同舍入模式下的舍入操作汇总**

![](/img/user/阅读库藏/assets/1659011344-c86eceadd88f962b6cdb52b5bbb8dbe2.png)

文章知识点与官方知识档案匹配，可进一步学习相关知识

[Java技能树](https://edu.csdn.net/skill/java/java-0593e0b9c9f74799a204d697f0db488b)[类和接口](https://edu.csdn.net/skill/java/java-0593e0b9c9f74799a204d697f0db488b)[类和面向对象](https://edu.csdn.net/skill/java/java-0593e0b9c9f74799a204d697f0db488b)44270 人正在系统学习中