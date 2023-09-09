---
{"title":"java - Spring MVC - Interceptor never called - Stack Overflow","url":"https://stackoverflow.com/questions/22633800/spring-mvc-interceptor-never-called","clipped_at":"2022-09-26 15:34:37","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/java-Spring-MVC-Interceptor-never-called-Stack-Overflow_1664177677/","dgPassFrontmatter":true}
---


# java - Spring MVC - Interceptor never called - Stack Overflow

# [Spring MVC - Interceptor never called](https://stackoverflow.com/questions/22633800/spring-mvc-interceptor-never-called)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 8 years, 6 months ago

Modified [8 months ago](https://stackoverflow.com/questions/22633800/spring-mvc-interceptor-never-called?lastactivity "2022-01-20 08:47:38Z")

Viewed 14k times

12

4

[](https://stackoverflow.com/posts/22633800/timeline)

I am trying to configure an interceptor in my application and I am not being able to make it work.

In my application configuration class, I have configured in the following way:

```java
@Configuration
@EnableWebMvc
public class AppContextConfiguration extends WebMvcConfigurerAdapter {
    ...
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor());
    }
    ...
}
```

And the interceptor:

```java
public class MyInterceptor extends HandlerInterceptorAdapter{

    private static final Logger logger = LoggerFactory.getLogger(MyInterceptor.class);

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, 
        Object handler) throws Exception {

        logger.debug("MyInterceptor - PREHANDLE");
    }
}
```

Does anybody know why is not being invoked?

*   [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")
*   [spring](https://stackoverflow.com/questions/tagged/spring "show questions tagged 'spring'")
*   [spring-mvc](https://stackoverflow.com/questions/tagged/spring-mvc "show questions tagged 'spring-mvc'")
*   [interceptor](https://stackoverflow.com/questions/tagged/interceptor "show questions tagged 'interceptor'")

[Share](https://stackoverflow.com/q/22633800 "Short permalink to this question")

[Improve this question](https://stackoverflow.com/posts/22633800/edit)

Follow

asked Mar 25, 2014 at 11:53

[

![r.rodriguez's user avatar](/img/user/阅读库藏/assets/1664177677-d7adfbc8dd0762bda8a4ef0a75085141.png)

](https://stackoverflow.com/users/1019997/r-rodriguez)

[r.rodriguez](https://stackoverflow.com/users/1019997/r-rodriguez)

64922 gold badges1010 silver badges2727 bronze badges

*   Put a log statement in `addInterceptors`. Is it logged?
    
    – [Sotirios Delimanolis](https://stackoverflow.com/users/438154/sotirios-delimanolis "266,972 reputation")
    
    [Mar 25, 2014 at 14:42](#comment34476472_22633800)
    
*   Also, your intercept won't currently compile. Show us what you actually have.
    
    – [Sotirios Delimanolis](https://stackoverflow.com/users/438154/sotirios-delimanolis "266,972 reputation")
    
    [Mar 25, 2014 at 14:48](#comment34476788_22633800)
    
*   Yes, if I put a log statement in addInterceptors it is logged, but the log statement in the preHandle method is never logged.
    
    – [r.rodriguez](https://stackoverflow.com/users/1019997/r-rodriguez "649 reputation")
    
    [Mar 26, 2014 at 11:12](#comment34514071_22633800)
    
*   You don't seem to have registered a path for the interceptor.
    
    – [Sotirios Delimanolis](https://stackoverflow.com/users/438154/sotirios-delimanolis "266,972 reputation")
    
    [Mar 26, 2014 at 11:28](#comment34514686_22633800)
    
*   1
    
    I tried with: **registry.addInterceptor(securityHandlerInterceptor()).addPathPatterns("/\* \*");** , **registry.addInterceptor(securityHandlerInterceptor()).addPathPatterns("/account");** and **registry.addInterceptor(securityHandlerInterceptor());** and it did not worked
    
    – [r.rodriguez](https://stackoverflow.com/users/1019997/r-rodriguez "649 reputation")
    
    [Mar 26, 2014 at 11:35](#comment34514927_22633800)
    

[Show **3** more comments](# "Expand to show all comments on this post")

## 7 Answers

Sorted by:

Highest score (default) Trending (recent votes count more) Date modified (newest first) Date created (oldest first)

31

[](https://stackoverflow.com/posts/35948730/timeline)

I'm using Spring Boot and was having the same problem where `addInterceptors()` was being called to register the interceptor, but the interceptor never fired during a request. Yet XML configuration worked no problem.

Basically, you don't need the `WebMvcConfigurerAdapter` class. You just need to declare an `@Bean` of type `MappedInterceptor`:

```java
@Bean
public MappedInterceptor myInterceptor()
{
    return new MappedInterceptor(null, new MyInterceptor());
}
```

[Share](https://stackoverflow.com/a/35948730 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/35948730/edit)

Follow

answered Mar 11, 2016 at 19:50

[

![Theo's user avatar](/img/user/阅读库藏/assets/1664177677-64f3b2796a9889d97465cbf01bbd3389.jpg)

](https://stackoverflow.com/users/6051625/theo)

[Theo](https://stackoverflow.com/users/6051625/theo)

32033 silver badges55 bronze badges

*   I am not sure what the underlying problem is, but this solved my problem. Thank you!
    
    – [Tamas](https://stackoverflow.com/users/322206/tamas "446 reputation")
    
    [Jan 31, 2017 at 7:47](#comment71083555_35948730)
    
*   @Theo do you have any idea why it's not working with registry.addInterceptor()?
    
    – [lucid](https://stackoverflow.com/users/7583619/lucid "2,502 reputation")
    
    [Apr 27, 2018 at 11:41](#comment87139546_35948730)
    
*   After a HUGE amount of researches, this is absolutely the only one RIGHT solution in order to configure an http interceptor on a old Spring MVC Framework (3 and 4).
    
    – [CptUnlucky](https://stackoverflow.com/users/9036081/cptunlucky "91 reputation")
    
    [Mar 2, 2021 at 9:49](#comment117451778_35948730)
    
*   @theo It worked bro!! Thanks for existing :)
    
    – [Vijay Rajpurohit](https://stackoverflow.com/users/8383549/vijay-rajpurohit "1,226 reputation")
    
    [Jun 10, 2021 at 19:18](#comment120062645_35948730)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

5

[](https://stackoverflow.com/posts/22634079/timeline)

Interceptor classes must be declared in _spring context xml_ configuration file within the tag `<mvc:interceptors>`. Did you do that?

From the Documentation

_An example of registering an interceptor applied to all URL paths:_

```java
<mvc:interceptors>
    <bean class="org.springframework.web.servlet.i18n.LocaleChangeInterceptor" />
</mvc:interceptors>
```

_An example of registering an interceptor limited to a specific URL path:_

```java
<mvc:interceptors>
    <mvc:interceptor>
        <mapping path="/secure/*"/>
        <bean class="org.example.SecurityInterceptor" />
    </mvc:interceptor>
</mvc:interceptors>
```

So, you would need to configure `MyInterceptor` class in the _spring context xml_ file

[Share](https://stackoverflow.com/a/22634079 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/22634079/edit)

Follow

answered Mar 25, 2014 at 12:05

[

![Keerthivasan's user avatar](/img/user/阅读库藏/assets/1664177677-bcedbefa0ac731c4664c83901db3b4d6.png)

](https://stackoverflow.com/users/2702504/keerthivasan)

[Keerthivasan](https://stackoverflow.com/users/2702504/keerthivasan)

12.5k22 gold badges3030 silver badges5252 bronze badges

*   2
    
    You can configure it as you said, via XML. But you can also configure via Java, and this is the way I am working in my application, trying to avoid the XML if is possible. However, the Java configuration should be as simple as the code I posted, but it does not work...
    
    – [r.rodriguez](https://stackoverflow.com/users/1019997/r-rodriguez "649 reputation")
    
    [Mar 25, 2014 at 12:21](#comment34470137_22634079)
    
*   Yes, I agree. What is the version of Spring you use? How did you confirm that it is not invoked?
    
    – [Keerthivasan](https://stackoverflow.com/users/2702504/keerthivasan "12,494 reputation")
    
    [Mar 25, 2014 at 12:27](#comment34470403_22634079)
    
*   I am using Spring 3.1.2, and I know that it is not being invoked because the log message is not being printed.
    
    – [r.rodriguez](https://stackoverflow.com/users/1019997/r-rodriguez "649 reputation")
    
    [Mar 25, 2014 at 12:32](#comment34470613_22634079)
    
*   ok, do you have component scan configured somewhere? `@ComponentScan(basePackages="webapp.base.package")`
    
    – [Keerthivasan](https://stackoverflow.com/users/2702504/keerthivasan "12,494 reputation")
    
    [Mar 25, 2014 at 12:36](#comment34470750_22634079)
    
*   Yes, I have. In fact, i had no problems with other beans which I have also configured via annotations, so this is mainly why I don't understand why it is not recognising the interceptor
    
    – [r.rodriguez](https://stackoverflow.com/users/1019997/r-rodriguez "649 reputation")
    
    [Mar 25, 2014 at 12:51](#comment34471379_22634079)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

3

[](https://stackoverflow.com/posts/52781681/timeline)

Can someone please mark Theos answer as the correct one? I had the situation of a perfectly working Spring Boot app using i18n and Thymeleaf (with a layout interceptor) as long as the app was running localhost with the following config:

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(localeChangeInterceptor());
    registry.addInterceptor(thymeleafLayoutInterceptor());
}
```

As soon as I deployed the app to an Elasticbeanstalk instance, both interceptors were not fired anymore. Although added once. When I changed the setting to

```java
@Bean
public MappedInterceptor localeInterceptor() {
    return new MappedInterceptor(null, localeChangeInterceptor());
}

@Bean
public MappedInterceptor thymeleafInterceptor() {
    return new MappedInterceptor(null, thymeleafLayoutInterceptor());
}
```

everything was working fine on all environments. There must be an issue with firing interceptors added with addInterceptor, it might depend on the URL that is used to invoke the request - I don't know.

Thanks for your answer, Theo, I just wanted to add this here if some else stumbles upon this nice feature.

[Share](https://stackoverflow.com/a/52781681 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/52781681/edit)

Follow

answered Oct 12, 2018 at 14:27

[

![Bruno's user avatar](/img/user/阅读库藏/assets/1664177677-beec9fd8a4dc8d1db9c360d8eca2d677.png)

](https://stackoverflow.com/users/9010171/bruno)

[Bruno](https://stackoverflow.com/users/9010171/bruno)

4355 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/39722617/timeline)

If it´s possible. Use this approach:

```java
public class Application extends WebMvcConfigurerAdapter{
...
@Bean
public MyInterceptor myInterceptor() {
    return new MyInterceptor();
}

public @Override void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(myInterceptor());
}
}
```

instead of:

```java
@Bean
public MappedInterceptor myInterceptor()
{
    return new MappedInterceptor(null, new MyInterceptor());
}
```

because with the first you can use injection features (like @Autowired, etc...)

[Share](https://stackoverflow.com/a/39722617 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/39722617/edit)

Follow

answered Sep 27, 2016 at 10:44

[

![Luis Costa's user avatar](/img/user/阅读库藏/assets/1664177677-bbd2aabf2ad461b423f95ac04370a0fd.png)

](https://stackoverflow.com/users/4147392/luis-costa)

[Luis Costa](https://stackoverflow.com/users/4147392/luis-costa)

1,09077 silver badges88 bronze badges

*   Not relevant to the question.
    
    – [user5365075](https://stackoverflow.com/users/5365075/user5365075 "1,914 reputation")
    
    [Mar 23, 2018 at 17:03](#comment85912730_39722617)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/59263706/timeline)

Maybe you should add componentscan annotation in the file where the main class is present.

```java
@ComponentScan("package where the interceptor is placed.")
```

Worked for me.

[Share](https://stackoverflow.com/a/59263706 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/59263706/edit)

Follow

answered Dec 10, 2019 at 9:11

[

![abhineet's user avatar](/img/user/阅读库藏/assets/1664177677-f36fc64495f79edca592a5894e312c83.png)

](https://stackoverflow.com/users/5649878/abhineet)

[abhineet](https://stackoverflow.com/users/5649878/abhineet)

1

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/66091813/timeline)

This approach worked with me

```java

@Configuration
public class WebConfiguration extends WebMvcConfigurerAdapter {
  @Override
  public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new MyInterceptor());
  }
}
```

[Share](https://stackoverflow.com/a/66091813 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/66091813/edit)

Follow

answered Feb 7, 2021 at 18:49

[

![jsmin's user avatar](/img/user/阅读库藏/assets/1664177677-9ae505f7cb10e9b600f49ac236f29738.png)

](https://stackoverflow.com/users/2737303/jsmin)

[jsmin](https://stackoverflow.com/users/2737303/jsmin)

34011 silver badge1111 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/70783146/timeline)

Using XML configuration, ensure you defined the interceptors in the correct context. Moving config from servlet context(\*-servlet) to main context (web.xml) made it work.

Even if the URL was a call to the servlet.

[Share](https://stackoverflow.com/a/70783146 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/70783146/edit)

Follow

answered Jan 20 at 8:47

[

![Michael Laffargue's user avatar](/img/user/阅读库藏/assets/1664177677-fa637d884463a67c2c5c590585bd76ac.jpg)

](https://stackoverflow.com/users/602928/michael-laffargue)

[Michael Laffargue](https://stackoverflow.com/users/602928/michael-laffargue)

9,97666 gold badges4040 silver badges7575 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

  

## Your Answer

### Sign up or [log in](https://stackoverflow.com/users/login?ssrc=question_page&returnurl=https%3a%2f%2fstackoverflow.com%2fquestions%2f22633800%2fspring-mvc-interceptor-never-called%23new-answer)

Sign up using Google

Sign up using Facebook

Sign up using Email and Password

 

### Post as a guest

Name

Email

Required, but never shown

Post Your Answer

By clicking “Post Your Answer”, you agree to our [terms of service](https://stackoverflow.com/legal/terms-of-service/public), [privacy policy](https://stackoverflow.com/legal/privacy-policy) and [cookie policy](https://stackoverflow.com/legal/cookie-policy)

## 

Not the answer you're looking for? Browse other questions tagged

*   [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")
*   [spring](https://stackoverflow.com/questions/tagged/spring "show questions tagged 'spring'")
*   [spring-mvc](https://stackoverflow.com/questions/tagged/spring-mvc "show questions tagged 'spring-mvc'")
*   [interceptor](https://stackoverflow.com/questions/tagged/interceptor "show questions tagged 'interceptor'")

or [ask your own question](https://stackoverflow.com/questions/ask).

*   The Overflow Blog
*   [Five nines uptime without developer burnout (Ep. 488)](https://stackoverflow.blog/2022/09/22/five-nines-uptime-without-developer-burnout/?cb=1)
    
*   [We hate Scrum and Agile…when it’s done wrong (Ep. 489)](https://stackoverflow.blog/2022/09/23/we-hate-scrum-and-agile-when-its-done-wrong-ep-488/?cb=1)
    
*   Featured on Meta
*   [Recent Color Contrast Changes and Accessibility Updates](https://meta.stackexchange.com/questions/382079/recent-color-contrast-changes-and-accessibility-updates?cb=1)
    
*   [Reviewer overboard! Or a request to improve the onboarding guidance for new...](https://meta.stackoverflow.com/questions/420349/reviewer-overboard-or-a-request-to-improve-the-onboarding-guidance-for-new-revi?cb=1 "Reviewer overboard! Or a request to improve the onboarding guidance for new reviewers in the suggested edits queue")
    
*   [Should I explain other people's code-only answers?](https://meta.stackoverflow.com/questions/416114/should-i-explain-other-peoples-code-only-answers?cb=1)
    

#### Linked

[

7

](https://stackoverflow.com/q/10745736?lq=1 "Question score (upvotes - downvotes)")[Spring 3.1 HandlerInterceptor Not being called](https://stackoverflow.com/questions/10745736/spring-3-1-handlerinterceptor-not-being-called?noredirect=1&lq=1)

[

3

](https://stackoverflow.com/q/49055909?lq=1 "Question score (upvotes - downvotes)")[spring-boot interceptor is not intercepting](https://stackoverflow.com/questions/49055909/spring-boot-interceptor-is-not-intercepting?noredirect=1&lq=1)

[

2

](https://stackoverflow.com/q/56798844?lq=1 "Question score (upvotes - downvotes)")[Java interceptor not getting called](https://stackoverflow.com/questions/56798844/java-interceptor-not-getting-called?noredirect=1&lq=1)

[

2

](https://stackoverflow.com/q/48306597?lq=1 "Question score (upvotes - downvotes)")[Spring MVC Interceptor pattern](https://stackoverflow.com/questions/48306597/spring-mvc-interceptor-pattern?noredirect=1&lq=1)

#### Related

[

572

](https://stackoverflow.com/q/3153546?rq=1 "Question score (upvotes - downvotes)")[How does autowiring work in Spring?](https://stackoverflow.com/questions/3153546/how-does-autowiring-work-in-spring?rq=1)

[

0

](https://stackoverflow.com/q/3343414?rq=1 "Question score (upvotes - downvotes)")[problem using UserService of google appengine](https://stackoverflow.com/questions/3343414/problem-using-userservice-of-google-appengine?rq=1)

[

423

](https://stackoverflow.com/q/3423262?rq=1 "Question score (upvotes - downvotes)")[What is @ModelAttribute in Spring MVC?](https://stackoverflow.com/questions/3423262/what-is-modelattribute-in-spring-mvc?rq=1)

[

2443

](https://stackoverflow.com/q/6827752?rq=1 "Question score (upvotes - downvotes)")[What's the difference between @Component, @Repository & @Service annotations in Spring?](https://stackoverflow.com/questions/6827752/whats-the-difference-between-component-repository-service-annotations-in?rq=1)

[

0

](https://stackoverflow.com/q/9154320?rq=1 "Question score (upvotes - downvotes)")[Gwt Controller With Annotation Approach](https://stackoverflow.com/questions/9154320/gwt-controller-with-annotation-approach?rq=1)

[

988

](https://stackoverflow.com/q/14014086?rq=1 "Question score (upvotes - downvotes)")[What is difference between CrudRepository and JpaRepository interfaces in Spring Data JPA?](https://stackoverflow.com/questions/14014086/what-is-difference-between-crudrepository-and-jparepository-interfaces-in-spring?rq=1)

[

445

](https://stackoverflow.com/q/16232833?rq=1 "Question score (upvotes - downvotes)")[How to respond with an HTTP 400 error in a Spring MVC @ResponseBody method returning String](https://stackoverflow.com/questions/16232833/how-to-respond-with-an-http-400-error-in-a-spring-mvc-responsebody-method-retur?rq=1)

[

392

](https://stackoverflow.com/q/16332092?rq=1 "Question score (upvotes - downvotes)")[Spring MVC @PathVariable with dot (.) is getting truncated](https://stackoverflow.com/questions/16332092/spring-mvc-pathvariable-with-dot-is-getting-truncated?rq=1)

[

992

](https://stackoverflow.com/q/21083170?rq=1 "Question score (upvotes - downvotes)")[How to configure port for a Spring Boot application](https://stackoverflow.com/questions/21083170/how-to-configure-port-for-a-spring-boot-application?rq=1)

[

0

](https://stackoverflow.com/q/56206143?rq=1 "Question score (upvotes - downvotes)")[Can't configure WebMvcConfigurer for interceptors addition in spring boot 2 application](https://stackoverflow.com/questions/56206143/cant-configure-webmvcconfigurer-for-interceptors-addition-in-spring-boot-2-appl?rq=1)

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [Why is the probability of a set of outcomes possibly non-zero if each outcome's probability is zero in continuous probability?](https://stats.stackexchange.com/questions/590041/why-is-the-probability-of-a-set-of-outcomes-possibly-non-zero-if-each-outcomes)
*   [What is the area of the SSME nozzle knowing only thrust at sea level and in vacuum?](https://space.stackexchange.com/questions/60485/what-is-the-area-of-the-ssme-nozzle-knowing-only-thrust-at-sea-level-and-in-vacu)
*   [What is the status of songs that glorify illegal activity in different countries?](https://law.stackexchange.com/questions/84638/what-is-the-status-of-songs-that-glorify-illegal-activity-in-different-countries)
*   [What is the smell after quartz is rubbed together?](https://physics.stackexchange.com/questions/729289/what-is-the-smell-after-quartz-is-rubbed-together)
*   [Is naturalism falsifiable?](https://philosophy.stackexchange.com/questions/93812/is-naturalism-falsifiable)
*   [The relation between s and jω](https://electronics.stackexchange.com/questions/636076/the-relation-between-s-and-j%cf%89)
*   [Are Gmail, Facebook and WhatsApp still accessible in Macau, and in Hong Kong in 2022?](https://travel.stackexchange.com/questions/176522/are-gmail-facebook-and-whatsapp-still-accessible-in-macau-and-in-hong-kong-in)
*   [What's that crown over Splatoon nametag players?](https://gaming.stackexchange.com/questions/399444/whats-that-crown-over-splatoon-nametag-players)
*   [How important is networking at conferences really?](https://academia.stackexchange.com/questions/189040/how-important-is-networking-at-conferences-really)
*   [Is there a strong correlation between credit score and mortgage rate in the US?](https://money.stackexchange.com/questions/152841/is-there-a-strong-correlation-between-credit-score-and-mortgage-rate-in-the-us)
*   [How can I get the value of a property at a specific key frame?](https://blender.stackexchange.com/questions/275407/how-can-i-get-the-value-of-a-property-at-a-specific-key-frame)
*   [When a cork is pulled out of a wine bottle, why does the inner end often expand more than the outer end?](https://physics.stackexchange.com/questions/728921/when-a-cork-is-pulled-out-of-a-wine-bottle-why-does-the-inner-end-often-expand)
*   [Is it okay to re-send a manuscript to another editor if there's no answer after three months?](https://academia.stackexchange.com/questions/189072/is-it-okay-to-re-send-a-manuscript-to-another-editor-if-theres-no-answer-after)
*   [formatting of sections and subsections](https://tex.stackexchange.com/questions/659473/formatting-of-sections-and-subsections)
*   [Homotopy type of Diff(ℝP³)](https://mathoverflow.net/questions/431204/homotopy-type-of-diff%e2%84%9dp%c2%b3)
*   [Carryless factors](https://codegolf.stackexchange.com/questions/252189/carryless-factors)
*   [Why does space warfare not lead to total anhiliation of both sides?](https://worldbuilding.stackexchange.com/questions/235999/why-does-space-warfare-not-lead-to-total-anhiliation-of-both-sides)
*   [How do I tape plastic to ceramic tiles?](https://diy.stackexchange.com/questions/257285/how-do-i-tape-plastic-to-ceramic-tiles)
*   [Installing outlet in electrical box with two sets of wires?](https://diy.stackexchange.com/questions/257309/installing-outlet-in-electrical-box-with-two-sets-of-wires)
*   [Are relative phase Toffoli gates universal for reversible circuits?](https://quantumcomputing.stackexchange.com/questions/28278/are-relative-phase-toffoli-gates-universal-for-reversible-circuits)
*   [Vagueness rather than specificity when the risks are enormous](https://politics.stackexchange.com/questions/75761/vagueness-rather-than-specificity-when-the-risks-are-enormous)
*   [SIngle-directional linked list in C](https://codereview.stackexchange.com/questions/279920/single-directional-linked-list-in-c)
*   [What is stronger: an ability that can be used proficiency bonus times, or one that can be used once per short rest?](https://rpg.stackexchange.com/questions/201646/what-is-stronger-an-ability-that-can-be-used-proficiency-bonus-times-or-one-th)
*   [Dimension too large for a simple 3D picture](https://tex.stackexchange.com/questions/659458/dimension-too-large-for-a-simple-3d-picture)

[Question feed](https://stackoverflow.com/feeds/question/22633800 "Feed of this question and its answers")