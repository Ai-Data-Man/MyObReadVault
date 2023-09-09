---
{"title":"聊聊Spring的环境抽象Environment，以及配置@Profile使用详解（介绍profile的6种激活方式）【享学Spring】","url":"https://blog.csdn.net/f641385712/article/details/94402262","clipped_at":"2022-09-23 16:32:41","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/聊聊Spring的环境抽象Environment-以及配置-Profile使用详解-介绍profile的6种激活方式-享学Spring_1663921961/","dgPassFrontmatter":true}
---


# 聊聊Spring的环境抽象Environment，以及配置@Profile使用详解（介绍profile的6种激活方式）【享学Spring】

#### 每篇一句

> 架构在于设计，设计在于细节。很多时候你的设计里有没有坑，就在于你对这些技术细节的把控

#### 前言

在我刚入行不久时，总是对上下文（`Context`）、环境（`Environment`）这类抽象概念搞不清楚、弄不明白、玩不转，更是不懂它哥俩的区别或者说是联系（说实话从中文上来说不好区分，至少我是这么认为的）。  
直到现在，我可以根据自己的理解对这两者下个通俗易懂的定义（不喜勿喷）：

*   **上下文**：用来处理分层传递的抽象，代表着应用
*   **环境**：当前上下文运行的环境，存储着各种**全局变量**。这些变量会影响着当前程序的运行情况（比如JDK信息、磁盘信息、内存信息等等。当然还包括用户自定义的一些属性值）

#### Spring属性管理API

其实在`Spring3.1`之前，在Spring中使用配置是有众多痛点的：比如**多环境支持**就是其中之一。直到`Spring3.1`，它提供了新的属性管理API，而且功能非常强大且很完善，以后对于一些属性信息（Properties）、配置信息（Profile）等都应该使用新的API来管理~

**核心API主要包括下面4个部分：**

1.  **PropertySource**：属性源。key-value属性对抽象
2.  **PropertyResolver**：属性解析器。用于解析相应key的value
3.  **Profile**：配置(资料里翻译为剖面，我实在不理解)。**只有激活的配置profile**的组件/配置才会注册到Spring容器，类似于maven中profile
4.  **Environment**：环境，本身也是个属性解析器`PropertyResolver`。它在基础上还提供了Profile特性，能够很好的对多环境支持。因此我们一般使用它，而不是底层接口`PropertyResolver`。 可以简单粗暴的把它理解为**Profile 和 PropertyResolver 的组合**

`Spring3.1`从这4个方面重新设计，使得API更加的清晰，而且功能也更加的强大了。

另外，关于`PropertySource`和`PropertyResolver`的说明，看官可先移步至：  
[【小家Spring】关于Spring属性处理器PropertyResolver以及应用运行环境Environment的深度分析，强大的StringValueResolver使用和解析](https://blog.csdn.net/f641385712/article/details/91380598)

#### 关于属性（Properties）

`PropertyResolver`作为属性解析器，用于解析**任何基础源**的属性的接口。它提供了最基本的`getProperty()`和`resolvePlaceholders()`等操作接口~~~

`ConfigurablePropertyResolver`作为`PropertyResolver`的子接口，**额外提供**属性类型转换的功能，说得通俗点就是在基础上提供了转换所需的`ConversionService`。

> 注意：接口上一直都没有提供`setProperty()`等方法，因为它的设计是通过属性源`PropertySource`来存储和获取属性值的，而不是一个简单的Map而已~

`AbstractPropertyResolver`是对接口`ConfigurablePropertyResolver`的抽象实现。比如它确定了前缀、后缀：

```java
public abstract class AbstractPropertyResolver implements ConfigurablePropertyResolver {
	...
	private String placeholderPrefix = SystemPropertyUtils.PLACEHOLDER_PREFIX; // ${
	private String placeholderSuffix = SystemPropertyUtils.PLACEHOLDER_SUFFIX; // }
	@Nullable
	private String valueSeparator = SystemPropertyUtils.VALUE_SEPARATOR; // :
	...
}
```

属性处理器在Spring内建**唯一实现类**为：`PropertySourcesPropertyResolver`。  
需要注意的是，环境抽象`Environment`关于属性部分的实现，也是委托给`PropertySourcesPropertyResolver`来处理的~

```java
public class PropertySourcesPropertyResolver extends AbstractPropertyResolver {
	...
	@Nullable
	private final PropertySources propertySources; //内部持有一组PropertySource

	// 由此可以看出propertySources的顺序很重要~~~
	// 并且还能处理占位符~~~~~ resolveNestedPlaceholders支持内嵌、嵌套占位符
	@Nullable
	protected <T> T getProperty(String key, Class<T> targetValueType, boolean resolveNestedPlaceholders) {
		if (this.propertySources != null) {
			for (PropertySource<?> propertySource : this.propertySources) {
				Object value = propertySource.getProperty(key);
				if (value != null) {
					if (resolveNestedPlaceholders && value instanceof String) {
						value = resolveNestedPlaceholders((String) value);
					}
					logKeyFound(key, propertySource, value);
					return convertValueIfNecessary(value, targetValueType);
				}
			}
		}
		return null;
	}
	...
}
```

`PropertySources`可以理解成一组`PropertySource`，它的唯一实现类为`MutablePropertySources`：

```java
public class MutablePropertySources implements PropertySources {
	// 内部确实是持有多个PropertySource
	private final List<PropertySource<?>> propertySourceList = new CopyOnWriteArrayList<>();

	public void addFirst(PropertySource<?> propertySource) {
		removeIfPresent(propertySource);
		this.propertySourceList.add(0, propertySource);
	}
	public void addLast(PropertySource<?> propertySource) {
		removeIfPresent(propertySource);
		this.propertySourceList.add(propertySource);
	}
	... 
}
```

#### 环境`Environment`和配置`Profile`

`Environment`表示当前应用程序正在运行的环境。  
`Environment`接口继承自`PropertyResolver`，所以它既能处理属性值、也能处理配置Profile：

1.  `properties`的属性值由`PropertyResolver`定义和处理
2.  `profile`则表示**当前的运行环境**配置（剖面），  
    对于应用程序中的 `properties` 而言，**并不是所有的都会加载到系统中**，只有其属性与 `profile` 匹配才会被激活加载

所以 [Environment](https://so.csdn.net/so/search?q=Environment&spm=1001.2101.3001.7020) 对象的作用是确定哪些配置文件（如果有）`profile` 当前处于活动状态，以及**默认情况下**哪些配置文件（如果有）应处于活动状态。

通过上面推荐博文里了解到了`PropertySource`，k-v属性值对系统**非常重要**且可以来自于多个地方：

*   自定义的Properties文件
*   JVM系统属性
*   系统环境变量
*   JNDI
*   ServeltContext、ServletConfig
*   …

`Environment`接口的定义如下：

```java
// @since 3.1
public interface Environment extends PropertyResolver {
	// 返回此环境下激活的配置文件集
	String[] getActiveProfiles();
	// 如果未设置激活配置文件，则返回默认的激活的配置文件集
	String[] getDefaultProfiles();

	// @since 5.1
	@Deprecated
	boolean acceptsProfiles(String... profiles);
	boolean acceptsProfiles(Profiles profiles);
}
```

###### ConfigurableEnvironment

提供设置激活的 `profile` 和默认的 `profile` 的功能以及操作 `Properties` 的工具（委托实现）。

```java
public interface ConfigurableEnvironment extends Environment, ConfigurablePropertyResolver {
    // 指定该环境下的 profile 集
    void setActiveProfiles(String... profiles);
    // 增加此环境的 profile
    void addActiveProfile(String profile);
    // 设置默认的 profile
    void setDefaultProfiles(String... profiles);
    // 返回此环境的 PropertySources
    MutablePropertySources getPropertySources();
   // 尝试返回 System.getenv() 的值，若失败则返回通过 System.getenv(string) 的来访问各个键的映射
    Map<String, Object> getSystemEnvironment();
    // 尝试返回 System.getProperties() 的值，若失败则返回通过 System.getProperties(string) 的来访问各个键的映射
    Map<String, Object> getSystemProperties();
    void merge(ConfigurableEnvironment parent);
}
```

该类除了继承 Environment 接口外还继承了 ConfigurablePropertyResolver 接口，所以它即具备了设置 profile 的功能也具备了操作 Properties 的功能。同时还允许客户端通过它设置和验证所需要的属性，自定义转换服务等功能。

###### AbstractEnvironment

它继承自`ConfigurableEnvironment`，它作为抽象实现，主要实现了`profile`相关功能。

```java
public abstract class AbstractEnvironment implements ConfigurableEnvironment {
	public static final String IGNORE_GETENV_PROPERTY_NAME = "spring.getenv.ignore";

	// 请参考：ConfigurableEnvironment#setActiveProfiles
	public static final String ACTIVE_PROFILES_PROPERTY_NAME = "spring.profiles.active";
	// 请参考：ConfigurableEnvironment#setDefaultProfiles
	public static final String DEFAULT_PROFILES_PROPERTY_NAME = "spring.profiles.default";


	private final Set<String> defaultProfiles = new LinkedHashSet<>(getReservedDefaultProfiles());
	// 默认的profile名称
	protected static final String RESERVED_DEFAULT_PROFILE_NAME = "default";
	...


	protected Set<String> doGetActiveProfiles() {
		synchronized (this.activeProfiles) {
			if (this.activeProfiles.isEmpty()) {
				String profiles = getProperty(ACTIVE_PROFILES_PROPERTY_NAME);
				if (StringUtils.hasText(profiles)) {
					setActiveProfiles(StringUtils.commaDelimitedListToStringArray(
							StringUtils.trimAllWhitespace(profiles)));
				}
			}
			return this.activeProfiles;
		}
	}
	...
}
```

如果 `activeProfiles` 为空，则从 `Properties` 中获取 `spring.profiles.active` 配置；如果不为空，则调用 `setActiveProfiles()` 设置 profile，最后返回。

> 从这里可以知道，API设置的activeProfiles优先级第一，其次才是属性配置（属性源是个很大的概念，希望各位看官能够理解）。

#### @Profile注解和`ProfileCondition`

上面介绍的都是API，其实Spring也非常友好的提供了注解的方式便捷的实现`Profile`能力：

```java
// @since 3.1  能标注在类上、方法上~
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {

	// 注意此处是个数组，也就是说可以激活多个的
	String[] value();

}
```

> 说明：spring-text包有个注解`org.springframework.test.context.ActiveProfiles`就是基于它的。

可以看到`@Profile`，从`Spring4.0`以后它的原理是基于`org.springframework.context.annotation.Condition`这个条件接口的，当然还少不了配套的`@Conditional`注解~~~

###### ProfileCondition

```java
class ProfileCondition implements Condition {

	@Override
	public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
		// 因为value值是个数组，所以此处有多个值 用的MultiValueMap
		MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
		if (attrs != null) {
			for (Object value : attrs.get("value")) {
		
				// 多个值中，但凡只要有一个acceptsProfiles了，那就返回true~
				if (context.getEnvironment().acceptsProfiles(Profiles.of((String[]) value))) {
					return true;
				}
			}
			return false;
		}
		return true;
	}

}
```

`@Profile`的value可以指定多个值，并且只需要有一个值符合了条件，`@Profile`标注的方法、类就会生效，就会被加入到容器内…

> 至于`@Conditional`是何时解析的，这部分知识在讲解Spring容器启动流程、初始化Bean的时候有N次提起，此处略

#### Profile使用的示例分析

在项目开发中，很多配置它在开发环境和线上环境是不一样的，最为典型就是**数据库连接**、`redis连接`等。  
**一般来说，最次都会有两种环境（公司越大、项目越复杂，环境会越多~）：**

1.  开发环境dev
2.  生产环境prod

本文就以这两个环境为基础，用一个**非常简单的例子**来演示`profile`的使用：

```java
@Configuration
public class RootConfig {

    @Profile({"default"}) // 显示的指出defualt~~~
    @Bean("person")
    public Person personDefault() {
        return new Person("我代表defualt数据源", 0);
    }

    @Profile({"dev", "test"})
    @Bean("person")
    public Person personDev() {
        return new Person("我代表dev数据源", 66);
    }


    @Profile({"prod", "online"})
    @Bean("person")
    public Person personProd() {
        return new Person("我代表prod数据源", 88);
    }

}
```

验证：

```java
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(RootConfig.class);

        Environment environment = context.getEnvironment();
        String[] activeProfiles = environment.getActiveProfiles();
        String[] defaultProfiles = environment.getDefaultProfiles();

        System.out.println(ArrayUtils.toString(activeProfiles)); //{}
        System.out.println(ArrayUtils.toString(defaultProfiles)); //{default}
        System.out.println(context.getBean("person"));

    }
```

结果打印：

```java
{}
{default}
Person{name='我代表defualt数据源', age=0}
```

由于我没有手动设置过任何激活Profiles，所以走默认的，因此最终被放进容器的Person对象就是我们那个`@Profile({"default"})`默认的。

**手动设置激活的Profiles：**

```java
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();

        // 手动设置激活的Profiles
        ConfigurableEnvironment environment = context.getEnvironment();
        environment.setActiveProfiles("dev", "test1", "test2");
        // environment.setDefaultProfiles("",""); // 也可以设置默认的  只是一般都不这么干~

        context.register(RootConfig.class);
        context.refresh();


        String[] activeProfiles = environment.getActiveProfiles();
        String[] defaultProfiles = environment.getDefaultProfiles();
        System.out.println(ArrayUtils.toString(activeProfiles)); //{}
        System.out.println(ArrayUtils.toString(defaultProfiles)); //{default}
        System.out.println(context.getBean("person"));

    }
```

结果打印：

```java
{dev,test1,test2}
{default}
Person{name='我代表dev数据源', age=66}
```

结果符合我们预期。这里需要注意的是：`environment.setActiveProfiles()`方法必须在容器启动之前调用，否则无效。  
（**激活`prod`的做法一样，此处就无需再演示了~~~**）

Tips：若类/方法上没有标注任何`@Profile`注解的，不受到activeProfiles的影响，都会被加入到容器。

* * *

* * *

#### 激活profile的6种方式

上面示例介绍的是自己手动**API调用方式**去激活profile，但在实际开发中，这样做显得非常的麻烦，而且并不是每位小伙伴都知道这个API和调用时机，**使用门槛偏高**。

Spring考虑到了这一点，所以它提供了非常多的方式让你可以激活profile，**任君灵活选择**。本文我介绍如下6种方式：

###### 方式一：API调用方式

见上例

###### 方式二：properties配置文件方式

写一个属性文件：`profile.properties`

```java
spring.profiles.active = prod
```

**把资源文件导入进容器**，然后启动即可

```java
@PropertySource("classpath:profile.properties")
@Configuration
public class RootConfig { ... }

    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(RootConfig.class);

        Environment environment = context.getEnvironment();
        String[] activeProfiles = environment.getActiveProfiles();
        String[] defaultProfiles = environment.getDefaultProfiles();


        System.out.println(ArrayUtils.toString(activeProfiles)); //{}
        System.out.println(ArrayUtils.toString(defaultProfiles)); //{default}
        System.out.println(context.getBean("person"));

    }
```

结果输出：

```java
{prod}
{default}
Person{name='我代表prod数据源', age=88}
```

注意：下面集中方式是`SpringBoot`环境下可直接使用的方式(Spring环境下默认不支持)：

###### 方式三：多profile文件方式

类似这么来写：

*   application-dev.properties
*   application-prod.properties

> 当然：类似的写多个yml也是可以的，此处就不作为一种新方式了

###### 方式四：yml文件`多文档块`方式

略

###### 方式五：命令行参数方式（重要）

形如：  
![在这里插入图片描述](/img/user/阅读库藏/assets/1663921961-e172a045377e2d2353b2e81fdc4bed83.png)

###### 方式六：JVM参数方式

形如：  
![在这里插入图片描述](/img/user/阅读库藏/assets/1663921961-6f87e85ef386adfea7fa02092c9b5471.png)

**思考题：若上面6中方式都有（SpringBoot环境下），或者只配置某几种方式，它们的优先级、最终谁会生效你知道吗？**

这个问题各位小伙伴可自行思考，此处我给出两点提示：

1.  API调用方式的优先第一考虑
2.  各式各样的`PropertySource`属性源的优先级才是应该考虑的重中之重

* * *

* * *

#### 总结

本篇文章讲述的内容还是非常非常简单的，核心是`Environment`它来管理`profile`。`@Profile`使用简单但功能强大，熟练的使用它很多时候都能让你达到**事半功倍**的效果。  
**了解了`Environment`以及`PropertySource`的联系以及编程模式，可以极大的实现程序的弹性。比如你可以抛弃Spring Cloud Config配置，自己采用第三方的集成**

最努力、最勤奋的人往往不是输出`"最多"`的，因为在人口众多且`效率为王`的当下，**只有越特殊越稀缺的人才，才越会被受到重视。**

* * *

# 关注A哥

| Author | [A哥(YourBatman)](https://www.yourbatman.cn/about) |
| --- | --- |
| 个人站点 | [www.yourbatman.cn](https://www.yourbatman.cn/) |
| E-mail | yourbatman@qq.com |
| **微 信** | [fsx641385712](https://www.yourbatman.cn/images/wechat.png) |
| **`活跃平台`** | [![](/img/user/阅读库藏/assets/1663921961-2ddb92c88596ac713e7517a583ed2508.png)](https://www.yourbatman.cn/) [![](/img/user/阅读库藏/assets/1663921961-ba019b6f43ac73e584ee8a54b5178ecf.png)](https://fangshixiang.blog.csdn.net/)[![](/img/user/阅读库藏/assets/1663921961-e025511ac6248d692a5361733a38cc7f.png)](https://yourbatman.cnblogs.com/)[![](/img/user/阅读库藏/assets/1663921961-de76ec508d255ac1a5ad90fa871dc072.png)](https://juejin.im/user/5b44d178f265da0fa1220efe/posts)[![](/img/user/阅读库藏/assets/1663921961-68f82c0a6bd8770b4d7532a3b1109869.png)](https://www.zhihu.com/people/fangshixiang/posts)[![](/img/user/阅读库藏/assets/1663921961-6f8d6a815161176890715195d9129e86.png)](https://www.jianshu.com/u/8740f1fdd684)[![](/img/user/阅读库藏/assets/1663921961-a52fca8435d7086688038179aa597f0c.png)](https://segmentfault.com/u/yourbatman/articles)[![](/img/user/阅读库藏/assets/1663921961-92f65cb266db69c22a1e114a8558041d.png)](https://github.com/yourbatman) |
| **公众号** | [BAT的乌托邦（ID：BAT-utopia）](https://www.yourbatman.cn/images/wechat_channel.jpg) |
| 知识星球 | [BAT的乌托邦](https://t.zsxq.com/nQBi66I) |
| 每日文章推荐 | [每日文章推荐](https://github.com/yourbatman/reading/issues) |

![BAT的乌托邦](/img/user/阅读库藏/assets/1663921961-82c08e1ef6d5e24eedb076c2f2ff0c10.gif)