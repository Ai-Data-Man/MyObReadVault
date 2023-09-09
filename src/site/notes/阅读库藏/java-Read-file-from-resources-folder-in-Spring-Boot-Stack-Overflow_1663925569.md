---
{"title":"java - Read file from resources folder in Spring Boot - Stack Overflow","url":"https://stackoverflow.com/questions/44399422/read-file-from-resources-folder-in-spring-boot","clipped_at":"2022-09-23 17:32:49","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/java-Read-file-from-resources-folder-in-Spring-Boot-Stack-Overflow_1663925569/","dgPassFrontmatter":true}
---


# java - Read file from resources folder in Spring Boot - Stack Overflow

# [Read file from resources folder in Spring Boot](https://stackoverflow.com/questions/44399422/read-file-from-resources-folder-in-spring-boot)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 5 years, 3 months ago

Modified [7 days ago](https://stackoverflow.com/questions/44399422/read-file-from-resources-folder-in-spring-boot?lastactivity "2022-09-15 11:16:19Z")

Viewed 391k times

166

32

[](https://stackoverflow.com/posts/44399422/timeline)

I'm using Spring Boot and `json-schema-validator`. I'm trying to read a file called `jsonschema.json` from the `resources` folder. I've tried a few different ways but I can't get it to work. This is my code.

```java
ClassLoader classLoader = getClass().getClassLoader();
File file = new File(classLoader.getResource("jsonschema.json").getFile());
JsonNode mySchema = JsonLoader.fromFile(file);
```

This is the location of the file.

[![enter image description here](/img/user/阅读库藏/assets/1663925569-36430efa55b726df851945090d8bc5bc.png)](https://i.stack.imgur.com/N3UzB.png)

And here I can see the file in the `classes` folder.

[![enter image description here](/img/user/阅读库藏/assets/1663925569-32edf73c0d5c89a0929fc1caa29f5411.png)](https://i.stack.imgur.com/waBk3.png)

But when I run the code I get the following error.

```java
jsonSchemaValidator error: java.io.FileNotFoundException: /home/user/Dev/Java/Java%20Programs/SystemRoutines/target/classes/jsonschema.json (No such file or directory)
```

What is it I'm doing wrong in my code?

*   [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")
*   [spring](https://stackoverflow.com/questions/tagged/spring "show questions tagged 'spring'")
*   [spring-boot](https://stackoverflow.com/questions/tagged/spring-boot "show questions tagged 'spring-boot'")
*   [json-schema-validator](https://stackoverflow.com/questions/tagged/json-schema-validator "show questions tagged 'json-schema-validator'")

[Share](https://stackoverflow.com/q/44399422 "Short permalink to this question")

[Improve this question](https://stackoverflow.com/posts/44399422/edit)

Follow

asked Jun 6, 2017 at 20:38

[

![g3blv's user avatar](/img/user/阅读库藏/assets/1663925569-157cac213311c028fac177bdbb3d3354.png)

](https://stackoverflow.com/users/2420986/g3blv)

[g3blv](https://stackoverflow.com/users/2420986/g3blv)

3,32766 gold badges3535 silver badges4747 bronze badges

*   Can you try this? `ClassLoader classLoader = getClass().getClassLoader(); JsonNode mySchema = JsonLoader.getJson(classLoader.getResourceAsStream("jsonschema.json"));`
    
    – [harshavmb](https://stackoverflow.com/users/3405980/harshavmb "3,189 reputation")
    
    [Jun 7, 2017 at 5:46](#comment75807896_44399422)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 24 Answers

Sorted by:

Highest score (default) Trending (recent votes count more) Date modified (newest first) Date created (oldest first)

141

[](https://stackoverflow.com/posts/49468282/timeline)

After spending a lot of time trying to resolve this issue, finally found a solution that works. The solution makes use of Spring's ResourceUtils. Should work for json files as well.

Thanks for the well written page by Lokesh Gupta : [Blog](https://howtodoinjava.com/core-java/io/read-file-from-resources-folder/)

[![enter image description here](/img/user/阅读库藏/assets/1663925569-39196a15f1cafe6652113b61f3c042c9.png)](https://i.stack.imgur.com/BUczm.png)

```java
package utils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.util.ResourceUtils;

import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;
import java.io.File;


public class Utils {

    private static final Logger LOGGER = LoggerFactory.getLogger(Utils.class.getName());

    public static Properties fetchProperties(){
        Properties properties = new Properties();
        try {
            File file = ResourceUtils.getFile("classpath:application.properties");
            InputStream in = new FileInputStream(file);
            properties.load(in);
        } catch (IOException e) {
            LOGGER.error(e.getMessage());
        }
        return properties;
    }
}
```

To answer a few concerns on the comments :

Pretty sure I had this running on Amazon EC2 using `java -jar target/image-service-slave-1.0-SNAPSHOT.jar`

Look at my github repo : [https://github.com/johnsanthosh/image-service](https://github.com/johnsanthosh/image-service) to figure out the right way to run this from a JAR.

[Share](https://stackoverflow.com/a/49468282 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/49468282/edit)

Follow

[edited Feb 4, 2020 at 11:13](https://stackoverflow.com/posts/49468282/revisions "show all edits to this post")

[

![Jochen van Wylick's user avatar](/img/user/阅读库藏/assets/1663925569-a22543065d862f8543b29f461d53848d.jpg)

](https://stackoverflow.com/users/896697/jochen-van-wylick)

[Jochen van Wylick](https://stackoverflow.com/users/896697/jochen-van-wylick)

5,14344 gold badges4040 silver badges6363 bronze badges

answered Mar 24, 2018 at 18:22

[

![John's user avatar](/img/user/阅读库藏/assets/1663925569-1c6d374f521199d803ccfbadd9d15e23.jpg)

](https://stackoverflow.com/users/2424981/john)

[John](https://stackoverflow.com/users/2424981/john)

2,09122 gold badges1616 silver badges2323 bronze badges

*   1
    
    Thanks John for adding this. This works and certainly a better approach using the ResourceUtil.
    
    – [Athar](https://stackoverflow.com/users/1254491/athar "569 reputation")
    
    [May 25, 2018 at 9:48](#comment88064270_49468282)
    
*   2
    
    @Athar Glad that I could help.
    
    – [John](https://stackoverflow.com/users/2424981/john "2,091 reputation")
    
    [Jun 22, 2018 at 17:01](#comment88979813_49468282)
    
*   61
    
    This will work only if you try to run the application from IDE but when you run the jar it won't find the file for you.
    
    – [Hassan Mudassir](https://stackoverflow.com/users/8854463/hassan-mudassir "171 reputation")
    
    [Aug 30, 2018 at 9:53](#comment91138175_49468282)
    
*   27
    
    Agree with Hassan, we should instead use `new ClassPathResource("filename").getInputStream()` if run the application from jar. [Detail](https://www.baeldung.com/spring-classpath-file-access)
    
    – [Jingchao Luan](https://stackoverflow.com/users/7112002/jingchao-luan "361 reputation")
    
    [Apr 3, 2019 at 22:34](#comment97716452_49468282)
    
*   3
    
    Agree with Hassan. As a caveat, ResourceUtils Javadoc is clear that the class is mainly for internal use. Check as well [stackoverflow.com/questions/25869428/…](https://stackoverflow.com/questions/25869428/classpath-resource-not-found-when-running-as-jar/25873705#25873705 "classpath resource not found when running as jar")
    
    – [Eric Jiang](https://stackoverflow.com/users/4138136/eric-jiang "134 reputation")
    
    [Apr 12, 2019 at 8:28](#comment97984011_49468282)
    

[Show **3** more comments](# "Expand to show all comments on this post")

69

[](https://stackoverflow.com/posts/44399541/timeline)

Very short answer: you are looking for the resource in the scope of a classloader's class instead of your target class. This should work:

```java
File file = new File(getClass().getResource("jsonschema.json").getFile());
JsonNode mySchema = JsonLoader.fromFile(file);
```

Also, that might be helpful reading:

*   [What is the difference between Class.getResource() and ClassLoader.getResource()?](https://stackoverflow.com/questions/6608795/what-is-the-difference-between-class-getresource-and-classloader-getresource)
*   [Strange behavior of Class.getResource() and ClassLoader.getResource() in executable jar](https://stackoverflow.com/questions/13269556/strange-behavior-of-class-getresource-and-classloader-getresource-in-executa)
*   [Loading resources using getClass().getResource()](https://stackoverflow.com/questions/2343187/loading-resources-using-getclass-getresource)

P.S. there is a case when a project compiled on one machine and after that launched on another or inside Docker. In such a scenario path to your resource folder would be invalid and you would need to get it in runtime:

```java
ClassPathResource res = new ClassPathResource("jsonschema.json");    
File file = new File(res.getPath());
JsonNode mySchema = JsonLoader.fromFile(file);
```

_**Update from 2020**_

On top of that if you want to read resource file as a String, for example in your tests, you can use these static utils methods:

```java
public static String getResourceFileAsString(String fileName) {
    InputStream is = getResourceFileAsInputStream(fileName);
    if (is != null) {
        BufferedReader reader = new BufferedReader(new InputStreamReader(is));
        return (String)reader.lines().collect(Collectors.joining(System.lineSeparator()));
    } else {
        throw new RuntimeException("resource not found");
    }
}

public static InputStream getResourceFileAsInputStream(String fileName) {
    ClassLoader classLoader = {CurrentClass}.class.getClassLoader();
    return classLoader.getResourceAsStream(fileName);
}
```

Example of usage:

```java
String soapXML = getResourceFileAsString("some_folder_in_resources/SOPA_request.xml");
```

[Share](https://stackoverflow.com/a/44399541 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/44399541/edit)

Follow

[edited Mar 26, 2021 at 14:50](https://stackoverflow.com/posts/44399541/revisions "show all edits to this post")

answered Jun 6, 2017 at 20:45

[

![Serhii Povísenko's user avatar](/img/user/阅读库藏/assets/1663925569-2a6372221bd238ce9a504c34bb98d1df.jpg)

](https://stackoverflow.com/users/2852528/serhii-pov%c3%adsenko)

[Serhii Povísenko](https://stackoverflow.com/users/2852528/serhii-pov%c3%adsenko)

2,98311 gold badge2626 silver badges4444 bronze badges

*   4
    
    `getClass().getResource("jsonschema.json")` returns `null`. I also tried `ClassPathResource res = new ClassPathResource("jsonschema.json")` which just returns `jsonschema.json`. Does this has something to do with that I'm using Spring Boot?
    
    – [g3blv](https://stackoverflow.com/users/2420986/g3blv "3,327 reputation")
    
    [Jun 7, 2017 at 5:15](#comment75807189_44399541)
    
*   @g3blv regarding `getClass().getResource("jsonschema.json")` returns `null` I could refer to this topic [stackoverflow.com/questions/26328040/…](https://stackoverflow.com/questions/26328040/intellij-idea-getclass-getresource-return-null/40065607 "intellij idea getclass getresource return null"). On top of that try to rebuild your project. Feedback would be appreciated.
    
    – [Serhii Povísenko](https://stackoverflow.com/users/2852528/serhii-pov%c3%adsenko "2,983 reputation")
    
    [Aug 23, 2019 at 13:15](#comment101707656_44399541)
    
*   @g3blv I provided update on the answer, please check
    
    – [Serhii Povísenko](https://stackoverflow.com/users/2852528/serhii-pov%c3%adsenko "2,983 reputation")
    
    [Feb 25, 2020 at 13:30](#comment106839777_44399541)
    
*   1
    
    @povisenko I would suggest you to throw exception if `is` empty. It means that the file/resource you are looking for is not there.
    
    – [endertunc](https://stackoverflow.com/users/1921974/endertunc "1,759 reputation")
    
    [May 17, 2020 at 14:21](#comment109401120_44399541)
    
*   1
    
    complete answer. Works both in the IDE and for the jar. Thanks.
    
    – [Digao](https://stackoverflow.com/users/2668689/digao "480 reputation")
    
    [Sep 2, 2020 at 21:28](#comment112667011_44399541)
    

[Show **1** more comment](# "Expand to show all comments on this post")

44

[](https://stackoverflow.com/posts/53076111/timeline)

if you have for example config folder under Resources folder I tried this Class working perfectly hope be useful

```java
File file = ResourceUtils.getFile("classpath:config/sample.txt")

//Read File Content
String content = new String(Files.readAllBytes(file.toPath()));
System.out.println(content);
```

[Share](https://stackoverflow.com/a/53076111 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/53076111/edit)

Follow

answered Oct 31, 2018 at 4:03

[

![Ismail's user avatar](/img/user/阅读库藏/assets/1663925569-060afc82fbf3549d30dd3e55ac56cf32.png)

](https://stackoverflow.com/users/3318976/ismail)

[Ismail](https://stackoverflow.com/users/3318976/ismail)

1,3621313 silver badges1010 bronze badges

*   14
    
    I tried your solution, it works in IDE but when you make spring jar input stream will help.
    
    – [Ameya](https://stackoverflow.com/users/353497/ameya "1,844 reputation")
    
    [Mar 20, 2019 at 4:48](#comment97239546_53076111)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

28

[](https://stackoverflow.com/posts/59569027/timeline)

Spent way too much time coming back to this page so just gonna leave this here:

```java
File file = new ClassPathResource("data/data.json").getFile();
```

[Share](https://stackoverflow.com/a/59569027 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/59569027/edit)

Follow

answered Jan 2, 2020 at 19:35

[

![Emmanuel Osimosu's user avatar](/img/user/阅读库藏/assets/1663925569-49e11095e006154ed2227dfa28111ad6.png)

](https://stackoverflow.com/users/4069782/emmanuel-osimosu)

[Emmanuel Osimosu](https://stackoverflow.com/users/4069782/emmanuel-osimosu)

5,21222 gold badges3636 silver badges3838 bronze badges

*   Works well in IDE but not in JAR-file.
    
    – [Kevin O.](https://stackoverflow.com/users/9390432/kevin-o "147 reputation")
    
    [Sep 13 at 23:56](#comment130161694_59569027)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

16

[](https://stackoverflow.com/posts/69269567/timeline)

## 2021 The Best Way

* * *

Simplest way to read file is:

```java
    Resource resource = new ClassPathResource("jsonSchema.json");
    FileInputStream file = new FileInputStream(resource.getFile());
```

[Share](https://stackoverflow.com/a/69269567 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/69269567/edit)

Follow

[edited Sep 21, 2021 at 13:50](https://stackoverflow.com/posts/69269567/revisions "show all edits to this post")

answered Sep 21, 2021 at 13:15

[

![Taha Malik's user avatar](/img/user/阅读库藏/assets/1663925569-b0be79d046fb5c1cda37d92b5cdb5fb5.png)

](https://stackoverflow.com/users/8647537/taha-malik)

[Taha Malik](https://stackoverflow.com/users/8647537/taha-malik)

1,83211 gold badge1414 silver badges2727 bronze badges

*   6
    
    This won't work in executable jar. Instead we can use `InputStream inputStream = resource.getInputStream();`
    
    – [srp](https://stackoverflow.com/users/8385477/srp "493 reputation")
    
    [Dec 1, 2021 at 6:39](#comment124059458_69269567)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

13

[](https://stackoverflow.com/posts/56854616/timeline)

See my answer here: [https://stackoverflow.com/a/56854431/4453282](https://stackoverflow.com/a/56854431/4453282)

```java
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
```

Use these 2 imports.

Declare

```java
@Autowired
ResourceLoader resourceLoader;
```

Use this in some function

```java
Resource resource=resourceLoader.getResource("classpath:preferences.json");
```

In your case, as you need the file you may use following

`File file = resource.getFile()`

Reference:[http://frugalisminds.com/spring/load-file-classpath-spring-boot/](http://frugalisminds.com/spring/load-file-classpath-spring-boot/) As already mentioned in previous answers don't use ResourceUtils it doesn't work after deployment of JAR, this will work in IDE as well as after deployment

[Share](https://stackoverflow.com/a/56854616 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/56854616/edit)

Follow

answered Jul 2, 2019 at 14:22

[

![sajal rajabhoj's user avatar](/img/user/阅读库藏/assets/1663925569-e38835ca3d3fce016df1467b9cdf0d02.jpg)

](https://stackoverflow.com/users/4453282/sajal-rajabhoj)

[sajal rajabhoj](https://stackoverflow.com/users/4453282/sajal-rajabhoj)

48944 silver badges1414 bronze badges

*   Which solution? I teseted it and its in PROD, not sure, you must be facing other issue.
    
    – [sajal rajabhoj](https://stackoverflow.com/users/4453282/sajal-rajabhoj "489 reputation")
    
    [Jan 25, 2021 at 9:28](#comment116483452_56854616)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

8

[](https://stackoverflow.com/posts/60397317/timeline)

Below is my working code.

```java
List<sampleObject> list = new ArrayList<>();
File file = new ClassPathResource("json/test.json").getFile();
ObjectMapper objectMapper = new ObjectMapper();
sampleObject = Arrays.asList(objectMapper.readValue(file, sampleObject[].class));
```

Hope it helps one!

[Share](https://stackoverflow.com/a/60397317 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/60397317/edit)

Follow

answered Feb 25, 2020 at 14:50

[

![Bhaumik Thakkar's user avatar](/img/user/阅读库藏/assets/1663925569-61ff4e58cdee85787802a45d7ff26fe9.png)

](https://stackoverflow.com/users/4192735/bhaumik-thakkar)

[Bhaumik Thakkar](https://stackoverflow.com/users/4192735/bhaumik-thakkar)

51811 gold badge99 silver badges2626 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

7

[](https://stackoverflow.com/posts/68913967/timeline)

### How to get resource reliably

To reliably get a file from the resources in Spring Boot application:

1.  Find a way to pass abstract resource, for example, `InputStream`, `URL` instead of `File`
2.  Use framework facilities to get the resource

## Example: read file from `resources`

```java
public class SpringBootResourcesApplication {
    public static void main(String[] args) throws Exception {
        ClassPathResource resource = new ClassPathResource("/hello", SpringBootResourcesApplication.class);
        try (InputStream inputStream = resource.getInputStream()) {
            String string = new String(inputStream.readAllBytes(), StandardCharsets.UTF_8);
            System.out.println(string);
        }
    }
}
```

*   [`ClassPathResource`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/ClassPathResource.html) is Spring's implementation of `Resource` - the abstract way to load _resource_. It is instantiated using the [`ClassPathResource(String, Class<?>)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/ClassPathResource.html#ClassPathResource-java.lang.String-java.lang.Class-) constructor:
    
    *   `/hello` is a path to the file
        *   The leading slash loads file by absolute path in classpath
            *   It is required because otherwise the path would be relative to the class
            *   If you pass a `ClassLoader` instead of `Class`, the slash can be omitted
            *   See also [What is the difference between Class.getResource() and ClassLoader.getResource()?](https://stackoverflow.com/questions/6608795/what-is-the-difference-between-class-getresource-and-classloader-getresource)
    *   The second argument is the `Class` to load the resource by
        *   Prefer passing the `Class` instead of `ClassLoader`, because [`ClassLoader.getResource` differs from `Class.getResource` in JPMS](https://stackoverflow.com/questions/48768879/how-to-access-resource-using-class-loader-in-java-9)
*   Project structure:
    
    ```java
    ├── mvnw
    ├── mvnw.cmd
    ├── pom.xml
    └── src
        └── main
            ├── java
            │   └── com
            │       └── caco3
            │           └── springbootresources
            │               └── SpringBootResourcesApplication.java
            └── resources
                ├── application.properties
                └── hello
    ```
    

The example above works from both IDE and jar

#### Deeper explanation

###### Prefer abstract resources instead of `File`

*   Examples of abstract resources are `InputStream` and `URL`
*   Avoid using `File` because it is not always possible to get it from a classpath resource
    
    *   E.g. the following code works in IDE:
    
    ```java
    public class SpringBootResourcesApplication {
        public static void main(String[] args) throws Exception {
            ClassLoader classLoader = SpringBootResourcesApplication.class.getClassLoader();
            File file = new File(classLoader.getResource("hello").getFile());
    
            Files.readAllLines(file.toPath(), StandardCharsets.UTF_8)
                    .forEach(System.out::println);
        }
    }
    ```
    
    but fails with:
    
    ```java
    java.nio.file.NoSuchFileException: file:/home/caco3/IdeaProjects/spring-boot-resources/target/spring-boot-resources-0.0.1-SNAPSHOT.jar!/BOOT-INF/classes!/hello
            at java.base/sun.nio.fs.UnixException.translateToIOException(UnixException.java:92)
            at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:111)
            at java.base/sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:116)
    ```
    
    when Spring Boot jar run
*   If you use external library, and it asks you for a resource, try to find a way to pass it an `InputStream` or `URL`
    *   For example the `JsonLoader.fromFile` from the question could be replaced with [`JsonLoader.fromURL`](http://javadox.com/com.github.fge/jackson-coreutils/1.8/com/github/fge/jackson/JsonLoader.html#fromURL(java.net.URL)) method: it accepts `URL`

###### Use framework's facilities to get the resource:

Spring Framework enables access to classpath resources through [`ClassPathResource`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/ClassPathResource.html)

You can use it:

1.  Directly, as in the example of reading file from `resources`
2.  Indirectly:
    1.  Using [`@Value`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Value.html):
        
        ```java
        @SpringBootApplication
        public class SpringBootResourcesApplication implements ApplicationRunner {
            @Value("classpath:/hello") // Do not use field injection
            private Resource resource;
        
            public static void main(String[] args) throws Exception {
                SpringApplication.run(SpringBootResourcesApplication.class, args);
            }
        
           @Override
           public void run(ApplicationArguments args) throws Exception {
               try (InputStream inputStream = resource.getInputStream()) {
                   String string = new String(inputStream.readAllBytes(), StandardCharsets.UTF_8);
                   System.out.println(string);
               }
           }
        }
        ```
        
    2.  Using [`ResourceLoader`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/ResourceLoader.html):
        
        ```java
        @SpringBootApplication
        public class SpringBootResourcesApplication implements ApplicationRunner {
            @Autowired // do not use field injection
            private ResourceLoader resourceLoader;
        
            public static void main(String[] args) throws Exception {
                SpringApplication.run(SpringBootResourcesApplication.class, args);
            }
        
            @Override
            public void run(ApplicationArguments args) throws Exception {
                Resource resource = resourceLoader.getResource("/hello");
                try (InputStream inputStream = resource.getInputStream()) {
                    String string = new String(inputStream.readAllBytes(), StandardCharsets.UTF_8);
                    System.out.println(string);
                }
            }
        }
        ```
        
        *   See also [this](https://stackoverflow.com/a/66986555/6486622) answer

[Share](https://stackoverflow.com/a/68913967 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/68913967/edit)

Follow

answered Aug 24, 2021 at 20:54

[

![Denis Zavedeev's user avatar](/img/user/阅读库藏/assets/1663925569-5d8377542f9b765cb40207281de20c84.png)

](https://stackoverflow.com/users/6486622/denis-zavedeev)

[Denis Zavedeev](https://stackoverflow.com/users/6486622/denis-zavedeev)

6,70444 gold badges3333 silver badges5050 bronze badges

*   ClassPathResource does not work in Fat jar
    
    – [user12384512](https://stackoverflow.com/users/359376/user12384512 "3,342 reputation")
    
    [Jan 27 at 19:35](#comment125313023_68913967)
    
*   Can I ask you to provide more details, please, maybe, you can post a simple application where it does not work?
    
    – [Denis Zavedeev](https://stackoverflow.com/users/6486622/denis-zavedeev "6,704 reputation")
    
    [Jan 27 at 21:22](#comment125315108_68913967)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

6

[](https://stackoverflow.com/posts/50654468/timeline)

create json folder in resources as subfolder then add json file in folder then you can use this code : [![enter image description here](/img/user/阅读库藏/assets/1663925569-8e1d462010b9e9e531124a5101d55f42.png)](https://i.stack.imgur.com/j44ZG.png)

`import com.fasterxml.jackson.core.type.TypeReference;`

`InputStream is = TypeReference.class.getResourceAsStream("/json/fcmgoogletoken.json");`

this works in Docker.

[Share](https://stackoverflow.com/a/50654468 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/50654468/edit)

Follow

answered Jun 2, 2018 at 7:14

[

![MostafaMashayekhi's user avatar](/img/user/阅读库藏/assets/1663925569-98a239d14e8d867c33a431c18f1d0f1f.jpg)

](https://stackoverflow.com/users/5184681/mostafamashayekhi)

[MostafaMashayekhi](https://stackoverflow.com/users/5184681/mostafamashayekhi)

24.8k33 gold badges1919 silver badges3838 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

6

[](https://stackoverflow.com/posts/54225276/timeline)

Here is my solution. May help someone;

It returns InputStream, but i assume you can read from it too.

```java
InputStream is = Thread.currentThread().getContextClassLoader().getResourceAsStream("jsonschema.json");
```

[Share](https://stackoverflow.com/a/54225276 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/54225276/edit)

Follow

answered Jan 16, 2019 at 20:58

[

![Erçin Akçay's user avatar](/img/user/阅读库藏/assets/1663925569-a0485d46869414c84795b5ef4b1acbc7.png)

](https://stackoverflow.com/users/1184571/er%c3%a7in-ak%c3%a7ay)

[Erçin Akçay](https://stackoverflow.com/users/1184571/er%c3%a7in-ak%c3%a7ay)

1,32533 gold badges1717 silver badges3434 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

5

[](https://stackoverflow.com/posts/48887939/timeline)

stuck in the same issue, this helps me

```java
URL resource = getClass().getClassLoader().getResource("jsonschema.json");
JsonNode jsonNode = JsonLoader.fromURL(resource);
```

[Share](https://stackoverflow.com/a/48887939 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/48887939/edit)

Follow

[edited May 21, 2020 at 11:25](https://stackoverflow.com/posts/48887939/revisions "show all edits to this post")

answered Feb 20, 2018 at 14:48

[

![Govind Singh's user avatar](/img/user/阅读库藏/assets/1663925569-62c0c5a5e679ab3e08d8941b280d7e96.jpg)

](https://stackoverflow.com/users/3196723/govind-singh)

[Govind Singh](https://stackoverflow.com/users/3196723/govind-singh)

14.9k1414 gold badges6868 silver badges102102 bronze badges

*   actually, this is almost the same as given answer for more details see here [stackoverflow.com/questions/14739550/…](https://stackoverflow.com/questions/14739550/difference-between-getclass-getclassloader-getresource-and-getclass-getres "difference between getclass getclassloader getresource and getclass getres")
    
    – [Serhii Povísenko](https://stackoverflow.com/users/2852528/serhii-pov%c3%adsenko "2,983 reputation")
    
    [Feb 23, 2018 at 14:27](#comment84906427_48887939)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

3

[](https://stackoverflow.com/posts/61920466/timeline)

The simplest method to bring a resource from the classpath in the resources directory parsed into a String is the following one liner.

**As a String(Using Spring Libraries):**

```java
         String resource = StreamUtils.copyToString(
                new ClassPathResource("resource.json").getInputStream(), defaultCharset());
```

This method uses the [StreamUtils](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/util/StreamUtils.html) utility and streams the file as an input stream into a String in a concise compact way.

If you want the file as a byte array you can use basic Java File I/O libraries:

**As a byte array(Using Java Libraries):**

```java
byte[] resource = Files.readAllBytes(Paths.get("/src/test/resources/resource.json"));
```

[Share](https://stackoverflow.com/a/61920466 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/61920466/edit)

Follow

[edited May 20, 2020 at 18:44](https://stackoverflow.com/posts/61920466/revisions "show all edits to this post")

answered May 20, 2020 at 18:37

[

![anataliocs's user avatar](/img/user/阅读库藏/assets/1663925569-b05b64afb42ff921295b3d3131167ed6.jpg)

](https://stackoverflow.com/users/555177/anataliocs)

[anataliocs](https://stackoverflow.com/users/555177/anataliocs)

10.1k66 gold badges5454 silver badges7070 bronze badges

*   java.io.FileNotFoundException: class path resource \[src/test/resources/html/registration.html\] cannot be opened because it does not exist
    
    – [skyho](https://stackoverflow.com/users/12093975/skyho "698 reputation")
    
    [Jul 4 at 10:00](#comment128682466_61920466)
    
*   It's right : InputStream resource = new ClassPathResource( "html/test.html").getInputStream(); String s = StreamUtils.copyToString(resource, StandardCharsets.UTF\_8);
    
    – [skyho](https://stackoverflow.com/users/12093975/skyho "698 reputation")
    
    [Jul 4 at 10:05](#comment128682592_61920466)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

3

[](https://stackoverflow.com/posts/66471173/timeline)

If you're using `spring` and `jackson` (most of the larger applications will), then use a simple oneliner:

`JsonNode json = new ObjectMapper().readTree(new ClassPathResource("filename").getFile());`

[Share](https://stackoverflow.com/a/66471173 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/66471173/edit)

Follow

answered Mar 4, 2021 at 8:25

[

![membersound's user avatar](/img/user/阅读库藏/assets/1663925569-ecb3c8846fd32ab2d9cb98ef4c0fe267.png)

](https://stackoverflow.com/users/1194415/membersound)

[membersound](https://stackoverflow.com/users/1194415/membersound)

76.2k166166 gold badges537537 silver badges10231023 bronze badges

*   this absolutely works!
    
    – [SGuru](https://stackoverflow.com/users/2420909/sguru "615 reputation")
    
    [Jun 1 at 15:47](#comment128012148_66471173)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

2

[](https://stackoverflow.com/posts/66986555/timeline)

Spring provides `ResourceLoader` which can be used to load files.

```java
@Autowired
ResourceLoader resourceLoader;


// path could be anything under resources directory
File loadDirectory(String path){
        Resource resource = resourceLoader.getResource("classpath:"+path); 
        try {
            return resource.getFile();
        } catch (IOException e) {
            log.warn("Issue with loading path {} as file", path);
        }
        return null;
 }
```

Referred to this [link](https://smarterco.de/java-load-file-from-classpath-in-spring-boot/).

[Share](https://stackoverflow.com/a/66986555 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/66986555/edit)

Follow

answered Apr 7, 2021 at 12:57

[

![rai.skumar's user avatar](/img/user/阅读库藏/assets/1663925569-e1ab10510954c1a262c71711a7e30b66.jpg)

](https://stackoverflow.com/users/1734279/rai-skumar)

[rai.skumar](https://stackoverflow.com/users/1734279/rai-skumar)

9,90566 gold badges3939 silver badges5555 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

1

[](https://stackoverflow.com/posts/60509303/timeline)

For me, the bug had two fixes.

1.  Xml file which was named as SAMPLE.XML which was causing even the below solution to fail when deployed to aws ec2. The fix was to rename it to new\_sample.xml and apply the solution given below.
2.  Solution approach [https://medium.com/@jonathan.henrique.smtp/reading-files-in-resource-path-from-jar-artifact-459ce00d2130](https://medium.com/@jonathan.henrique.smtp/reading-files-in-resource-path-from-jar-artifact-459ce00d2130)

I was using Spring boot as jar and deployed to aws ec2 Java variant of the solution is as below :

```java
package com.test;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.stream.Collectors;
import java.util.stream.Stream;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.core.io.Resource;


public class XmlReader {

    private static Logger LOGGER = LoggerFactory.getLogger(XmlReader.class);

  public static void main(String[] args) {


      String fileLocation = "classpath:cbs_response.xml";
      String reponseXML = null;
      try (ClassPathXmlApplicationContext appContext = new ClassPathXmlApplicationContext()){

        Resource resource = appContext.getResource(fileLocation);
        if (resource.isReadable()) {
          BufferedReader reader =
              new BufferedReader(new InputStreamReader(resource.getInputStream()));
          Stream<String> lines = reader.lines();
          reponseXML = lines.collect(Collectors.joining("\n"));

        }      
      } catch (IOException e) {
        LOGGER.error(e.getMessage(), e);
      }
  }
}
```

[Share](https://stackoverflow.com/a/60509303 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/60509303/edit)

Follow

answered Mar 3, 2020 at 14:22

[

![Vijayakumar S's user avatar](/img/user/阅读库藏/assets/1663925569-4e14553e2ff597d3c4dc77b8fd05f72c.jpg)

](https://stackoverflow.com/users/4069023/vijayakumar-s)

[Vijayakumar S](https://stackoverflow.com/users/4069023/vijayakumar-s)

4544 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

1

[](https://stackoverflow.com/posts/67320729/timeline)

If you are using maven resource filter in your proyect, you need to configure what kind of file is going to be loaded in pom.xml. If you don't, no matter what class you choose to load the resource, it won't be found.

pom.xml

```java
<resources>
    <resource>
        <directory>${project.basedir}/src/main/resources</directory>
        <filtering>true</filtering>
        <includes>
            <include>**/*.properties</include>
            <include>**/*.yml</include>
            <include>**/*.yaml</include>
            <include>**/*.json</include>
        </includes>
    </resource>
</resources>
```

[Share](https://stackoverflow.com/a/67320729 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/67320729/edit)

Follow

answered Apr 29, 2021 at 15:42

[

![J. Escribano's user avatar](/img/user/阅读库藏/assets/1663925569-0e9623ba817571dbb9242869732ef237.jpg)

](https://stackoverflow.com/users/12851112/j-escribano)

[J. Escribano](https://stackoverflow.com/users/12851112/j-escribano)

3377 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

1

[](https://stackoverflow.com/posts/67394037/timeline)

Below works in both IDE and running it as a jar in the terminal,

```java
import org.springframework.core.io.Resource;

@Value("classpath:jsonschema.json")
Resource schemaFile;
    
JsonSchemaFactory factory = JsonSchemaFactory.getInstance(SpecVersion.VersionFlag.V4);
JsonSchema jsonSchema = factory.getSchema(schemaFile.getInputStream());
```

[Share](https://stackoverflow.com/a/67394037 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/67394037/edit)

Follow

answered May 5, 2021 at 1:22

[

![Pons's user avatar](/img/user/阅读库藏/assets/1663925569-7ad9b1ba8196a8f9dbb3c4d001f05111.png)

](https://stackoverflow.com/users/6506261/pons)

[Pons](https://stackoverflow.com/users/6506261/pons)

1,09111 gold badge1010 silver badges2020 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

1

[](https://stackoverflow.com/posts/71764431/timeline)

You need to sanitize the path and replace %20 with a space, or rename your directory. Then it should work.

```java
FileNotFoundException: /home/user/Dev/Java/Java%20Programs/SystemRoutines/target/classes/jsonschema.json
```

[Share](https://stackoverflow.com/a/71764431 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/71764431/edit)

Follow

answered Apr 6 at 9:40

[

![Andrei's user avatar](/img/user/阅读库藏/assets/1663925569-9387267a0804c1b4ad4e3e4b75c06cbd.png)

](https://stackoverflow.com/users/18723401/andrei)

[Andrei](https://stackoverflow.com/users/18723401/andrei)

1122 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

1

[](https://stackoverflow.com/posts/71767296/timeline)

just to add my solution as another 2 cents together with all other answers. I am using the Spring [DefaultResourceLoader](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/DefaultResourceLoader.html) to get a [ResourceLoader](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/ResourceLoader.html). Then the Spring [FileCopyUtils](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/util/FileCopyUtils.html) to get the content of the resource file to a string.

```java
import static java.nio.charset.StandardCharsets.UTF_8;

import java.io.IOException;
import java.io.InputStreamReader;
import java.io.Reader;
import java.io.UncheckedIOException;

import org.springframework.core.io.DefaultResourceLoader;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.util.FileCopyUtils;

public class ResourceReader {
    public static String readResourceFile(String path) {
        ResourceLoader resourceLoader = new DefaultResourceLoader();
        Resource resource = resourceLoader.getResource(path);
        return asString(resource);
    }

    private static String asString(Resource resource) {
        try (Reader reader = new InputStreamReader(resource.getInputStream(), UTF_8)) {
            return FileCopyUtils.copyToString(reader);
        } catch (IOException e) {
            throw new UncheckedIOException(e);
        }
    }
}
```

[Share](https://stackoverflow.com/a/71767296 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/71767296/edit)

Follow

answered Apr 6 at 13:04

[

![Knoblauch's user avatar](/img/user/阅读库藏/assets/1663925569-b7e339150fa4a6f6e8c99ea5acee3b82.jpg)

](https://stackoverflow.com/users/2096986/knoblauch)

[Knoblauch](https://stackoverflow.com/users/2096986/knoblauch)

6,03566 gold badges3636 silver badges8585 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

1

[](https://stackoverflow.com/posts/71781975/timeline)

Here is a solution with `ResourceUtils` and Java 11 `Files.readString` which takes care of UTF-8 encoding and resource closing

```java
import org.json.JSONObject;
import org.springframework.util.ResourceUtils;
 
public JSONObject getJsonData() throws IOException {
        //file path : src/main/resources/assets/data.json
        File file = ResourceUtils.getFile("classpath:assets/data.json");
        String data = Files.readString(file.toPath());
        return new JSONObject(data);
 }
```

But after deploying the application on OpenShift, the resource is not reachable. So the correct solution is

```java
import static java.nio.charset.StandardCharsets.UTF_8;
import static org.springframework.util.FileCopyUtils.copyToByteArray;
import org.springframework.core.io.ClassPathResource;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

public  JsonNode getJsonData() throws IOException {
  ClassPathResource classPathResource = new 
  ClassPathResource("assets/data.json");
        byte[] byteArray = 
  copyToByteArray(classPathResource.getInputStream());
        return new ObjectMapper() //
                .readTree(new String(byteArray, UTF_8));
}
```

[Share](https://stackoverflow.com/a/71781975 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/71781975/edit)

Follow

[edited Apr 12 at 7:11](https://stackoverflow.com/posts/71781975/revisions "show all edits to this post")

answered Apr 7 at 12:13

[

![Jafar Karuthedath's user avatar](/img/user/阅读库藏/assets/1663925569-ad37314095265ecdd66ec17839db976f.jpg)

](https://stackoverflow.com/users/2668564/jafar-karuthedath)

[Jafar Karuthedath](https://stackoverflow.com/users/2668564/jafar-karuthedath)

2,9972929 silver badges2020 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/63805315/timeline)

i think the problem lies within the space in the folder-name where your project is placed. /home/user/Dev/Java/Java%20Programs/SystemRoutines/target/classes/jsonschema.json

there is space between Java Programs.Renaming the folder name should make it work

[Share](https://stackoverflow.com/a/63805315 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/63805315/edit)

Follow

answered Sep 9, 2020 at 5:34

[

![sam's user avatar](/img/user/阅读库藏/assets/1663925569-59ccada0dd5d0271da9010fdd6a603a0.png)

](https://stackoverflow.com/users/7808334/sam)

[sam](https://stackoverflow.com/users/7808334/sam)

1533 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/68579909/timeline)

Using Spring **ResourceUtils.getFile()** you don't have to take care absolute path :)

```java
 private String readDictionaryAsJson(String filename) throws IOException {
    String fileContent;
    try {
        File file = ResourceUtils.getFile("classpath:" + filename);
        Path path = file.toPath();
        Stream<String> lines = Files.lines(path);
        fileContent = lines.collect(Collectors.joining("\n"));
    } catch (IOException ex) {
        throw ex;
    }
    return new fileContent;
}
```

[Share](https://stackoverflow.com/a/68579909 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/68579909/edit)

Follow

[edited Jul 29, 2021 at 16:55](https://stackoverflow.com/posts/68579909/revisions "show all edits to this post")

answered Jul 29, 2021 at 16:44

[

![szachMati's user avatar](/img/user/阅读库藏/assets/1663925569-b13d2c79fe69feeee3a328084bb2b903.png)

](https://stackoverflow.com/users/10736451/szachmati)

[szachMati](https://stackoverflow.com/users/10736451/szachmati)

16611 silver badge99 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/69226259/timeline)

Try this:

In application.properties

```java
app.jsonSchema=classpath:jsonschema.json
```

On your Properties pojo:

**NOTE**: You can use any prefered way of reading configs from application.properties.

```java
@Configuration
@ConfigurationProperties(prefix = "app") 
public class ConfigProperties {
private Resource jsonSchema;

// standard getters and setters
}
```

In your class, read the resource from the Properties Pojo:

```java
//Read the Resource and get the Input Stream
try (InputStream inStream = configProperties.getJsonSchema().getInputStream()) {
   //From here you can manipulate the Input Stream as desired....
   //Map the Input Stream to a Map
    ObjectMapper mapper = new ObjectMapper();
    Map <String, Object> jsonMap = mapper.readValue(inStream, Map.class);
    //Convert the Map to a JSON obj
    JSONObject json = new JSONObject(jsonMap);
    } catch (Exception e) {
        e.printStackTrace();
    }
```

[Share](https://stackoverflow.com/a/69226259 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/69226259/edit)

Follow

answered Sep 17, 2021 at 15:50

[

![Sylvester's user avatar](/img/user/阅读库藏/assets/1663925569-4b4efa61f166ea163014eb75763165ed.png)

](https://stackoverflow.com/users/10217900/sylvester)

[Sylvester](https://stackoverflow.com/users/10217900/sylvester)

1391212 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

\-2

[](https://stackoverflow.com/posts/68172886/timeline)

I had same issue and because I just had to get file path to send to file input stream, I did this way.

```java
    String pfxCertificate ="src/main/resources/cert/filename.pfx";
    String pfxPassword = "1234";
    FileInputStream fileInputStream = new FileInputStream(pfxCertificate));
```

[Share](https://stackoverflow.com/a/68172886 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/68172886/edit)

Follow

answered Jun 29, 2021 at 5:03

[

![Kiran Maharjan's user avatar](/img/user/阅读库藏/assets/1663925569-08ef47bcec68b5f5f6822a7b8c5c5026.jpg)

](https://stackoverflow.com/users/7473616/kiran-maharjan)

[Kiran Maharjan](https://stackoverflow.com/users/7473616/kiran-maharjan)

5333 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

  

## Your Answer

### Sign up or [log in](https://stackoverflow.com/users/login?ssrc=question_page&returnurl=https%3a%2f%2fstackoverflow.com%2fquestions%2f44399422%2fread-file-from-resources-folder-in-spring-boot%23new-answer)

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
*   [spring-boot](https://stackoverflow.com/questions/tagged/spring-boot "show questions tagged 'spring-boot'")
*   [json-schema-validator](https://stackoverflow.com/questions/tagged/json-schema-validator "show questions tagged 'json-schema-validator'")

or [ask your own question](https://stackoverflow.com/questions/ask).

*   The Overflow Blog
*   [A serial entrepreneur finally embraces open source (Ep. 486)](https://stackoverflow.blog/2022/09/20/a-serial-entrepreneur-finally-embraces-open-source-ep-486/?cb=1)
    
*   [Five nines uptime without developer burnout](https://stackoverflow.blog/2022/09/22/five-nines-uptime-without-developer-burnout/?cb=1)
    
*   Featured on Meta
*   [Recent Color Contrast Changes and Accessibility Updates](https://meta.stackexchange.com/questions/382079/recent-color-contrast-changes-and-accessibility-updates?cb=1)
    
*   [Reviewer overboard! Or a request to improve the onboarding guidance for new...](https://meta.stackoverflow.com/questions/420349/reviewer-overboard-or-a-request-to-improve-the-onboarding-guidance-for-new-revi?cb=1 "Reviewer overboard! Or a request to improve the onboarding guidance for new reviewers in the suggested edits queue")
    
*   [Should I explain other people's code-only answers?](https://meta.stackoverflow.com/questions/416114/should-i-explain-other-peoples-code-only-answers?cb=1)
    

#### Linked

[

436

](https://stackoverflow.com/q/5673260?lq=1 "Question score (upvotes - downvotes)")[Downloading a file from spring controllers](https://stackoverflow.com/questions/5673260/downloading-a-file-from-spring-controllers?noredirect=1&lq=1)

[

181

](https://stackoverflow.com/q/25869428?lq=1 "Question score (upvotes - downvotes)")[Classpath resource not found when running as jar](https://stackoverflow.com/questions/25869428/classpath-resource-not-found-when-running-as-jar?noredirect=1&lq=1)

[

225

](https://stackoverflow.com/q/6608795?lq=1 "Question score (upvotes - downvotes)")[What is the difference between Class.getResource() and ClassLoader.getResource()?](https://stackoverflow.com/questions/6608795/what-is-the-difference-between-class-getresource-and-classloader-getresource?noredirect=1&lq=1)

[

43

](https://stackoverflow.com/q/26328040?lq=1 "Question score (upvotes - downvotes)")[IntelliJ IDEA - getClass().getResource("...") return null](https://stackoverflow.com/questions/26328040/intellij-idea-getclass-getresource-return-null?noredirect=1&lq=1)

[

70

](https://stackoverflow.com/q/14739550?lq=1 "Question score (upvotes - downvotes)")[Difference between getClass().getClassLoader().getResource() and getClass.getResource()?](https://stackoverflow.com/questions/14739550/difference-between-getclass-getclassloader-getresource-and-getclass-getres?noredirect=1&lq=1)

[

26

](https://stackoverflow.com/q/2343187?lq=1 "Question score (upvotes - downvotes)")[Loading resources using getClass().getResource()](https://stackoverflow.com/questions/2343187/loading-resources-using-getclass-getresource?noredirect=1&lq=1)

[

39

](https://stackoverflow.com/q/13269556?lq=1 "Question score (upvotes - downvotes)")[Strange behavior of Class.getResource() and ClassLoader.getResource() in executable jar](https://stackoverflow.com/questions/13269556/strange-behavior-of-class-getresource-and-classloader-getresource-in-executa?noredirect=1&lq=1)

[

5

](https://stackoverflow.com/q/48768879?lq=1 "Question score (upvotes - downvotes)")[How to access resource using class loader in Java 9](https://stackoverflow.com/questions/48768879/how-to-access-resource-using-class-loader-in-java-9?noredirect=1&lq=1)

[

3

](https://stackoverflow.com/q/54053603?lq=1 "Question score (upvotes - downvotes)")[Spring boot can not find resource file after packaging](https://stackoverflow.com/questions/54053603/spring-boot-can-not-find-resource-file-after-packaging?noredirect=1&lq=1)

[

6

](https://stackoverflow.com/q/56042444?lq=1 "Question score (upvotes - downvotes)")[read file from jar in spring-boot](https://stackoverflow.com/questions/56042444/read-file-from-jar-in-spring-boot?noredirect=1&lq=1)

[See more linked questions](https://stackoverflow.com/questions/linked/44399422?lq=1)

#### Related

[

1713

](https://stackoverflow.com/q/326390?rq=1 "Question score (upvotes - downvotes)")[How do I create a Java string from the contents of a file?](https://stackoverflow.com/questions/326390/how-do-i-create-a-java-string-from-the-contents-of-a-file?rq=1)

[

768

](https://stackoverflow.com/q/1844688?rq=1 "Question score (upvotes - downvotes)")[How to read all files in a folder from Java?](https://stackoverflow.com/questions/1844688/how-to-read-all-files-in-a-folder-from-java?rq=1)

[

1027

](https://stackoverflow.com/q/11461607?rq=1 "Question score (upvotes - downvotes)")[Can't start Eclipse - Java was started but returned exit code=13](https://stackoverflow.com/questions/11461607/cant-start-eclipse-java-was-started-but-returned-exit-code-13?rq=1)

[

992

](https://stackoverflow.com/q/21083170?rq=1 "Question score (upvotes - downvotes)")[How to configure port for a Spring Boot application](https://stackoverflow.com/questions/21083170/how-to-configure-port-for-a-spring-boot-application?rq=1)

[

113

](https://stackoverflow.com/q/25764459?rq=1 "Question score (upvotes - downvotes)")[Spring Boot application.properties value not populating](https://stackoverflow.com/questions/25764459/spring-boot-application-properties-value-not-populating?rq=1)

[

475

](https://stackoverflow.com/q/30118683?rq=1 "Question score (upvotes - downvotes)")[How can I log SQL statements in Spring Boot?](https://stackoverflow.com/questions/30118683/how-can-i-log-sql-statements-in-spring-boot?rq=1)

[

16

](https://stackoverflow.com/q/32278204?rq=1 "Question score (upvotes - downvotes)")[specify files in resources folder in spring application.properties file](https://stackoverflow.com/questions/32278204/specify-files-in-resources-folder-in-spring-application-properties-file?rq=1)

[

1

](https://stackoverflow.com/q/56381512?rq=1 "Question score (upvotes - downvotes)")[Spring Maven reading from resources folder](https://stackoverflow.com/questions/56381512/spring-maven-reading-from-resources-folder?rq=1)

[

1

](https://stackoverflow.com/q/63355006?rq=1 "Question score (upvotes - downvotes)")[store file in spring boot resource folder after deployment](https://stackoverflow.com/questions/63355006/store-file-in-spring-boot-resource-folder-after-deployment?rq=1)

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [Combining a list with a certain index of a list](https://mathematica.stackexchange.com/questions/273781/combining-a-list-with-a-certain-index-of-a-list)
*   [Notes not adding up to time signature, with weird white oval note](https://music.stackexchange.com/questions/125043/notes-not-adding-up-to-time-signature-with-weird-white-oval-note)
*   [Stabilization of taking presheaf categories](https://mathoverflow.net/questions/431031/stabilization-of-taking-presheaf-categories)
*   [How is that there are two different words, comparo, that appear to be identical?](https://latin.stackexchange.com/questions/18861/how-is-that-there-are-two-different-words-comparo-that-appear-to-be-identical)
*   [How to properly acknowledge an old famous mathematician in my article?](https://academia.stackexchange.com/questions/188935/how-to-properly-acknowledge-an-old-famous-mathematician-in-my-article)
*   [Why would a society be matriarchal but patrilinear?](https://worldbuilding.stackexchange.com/questions/235922/why-would-a-society-be-matriarchal-but-patrilinear)
*   [Is there an algorithm for the genus of a knot?](https://mathoverflow.net/questions/430887/is-there-an-algorithm-for-the-genus-of-a-knot)
*   [Can professors arrange for an MSc scholarship if you contact them by e-mail and they are interested in working with you?](https://academia.stackexchange.com/questions/188975/can-professors-arrange-for-an-msc-scholarship-if-you-contact-them-by-e-mail-and)
*   [AWK check date if today then filter only that content](https://unix.stackexchange.com/questions/718295/awk-check-date-if-today-then-filter-only-that-content)
*   [How does SQL Server update lock work?](https://dba.stackexchange.com/questions/317240/how-does-sql-server-update-lock-work)
*   [Have anti-Ukraine-war parties in Russia explained what their preferred end state looks like?](https://politics.stackexchange.com/questions/75669/have-anti-ukraine-war-parties-in-russia-explained-what-their-preferred-end-state)
*   [Callout labels are wrong in GeoListPlot](https://mathematica.stackexchange.com/questions/273827/callout-labels-are-wrong-in-geolistplot)
*   [Is releasing this company's documents illegal?](https://law.stackexchange.com/questions/84605/is-releasing-this-companys-documents-illegal)
*   [How to address team member who excessively defers to me and is unwilling to give clear feedback](https://workplace.stackexchange.com/questions/187517/how-to-address-team-member-who-excessively-defers-to-me-and-is-unwilling-to-give)
*   [Possible explanations for abnormal day/night cycle?](https://worldbuilding.stackexchange.com/questions/235894/possible-explanations-for-abnormal-day-night-cycle)
*   [How can I get the R of this diagram with circutikz?](https://tex.stackexchange.com/questions/659162/how-can-i-get-the-r-of-this-diagram-with-circutikz)
*   [Where is the reaction force of an object that has just flung out from a circular motion?](https://physics.stackexchange.com/questions/728648/where-is-the-reaction-force-of-an-object-that-has-just-flung-out-from-a-circular)
*   [Can I replace 26 inch tires with 29 inch tires on my Giant Sedona?](https://bicycles.stackexchange.com/questions/85967/can-i-replace-26-inch-tires-with-29-inch-tires-on-my-giant-sedona)
*   [Why do we need the “Program Files” folder in Windows?](https://superuser.com/questions/1743822/why-do-we-need-the-program-files-folder-in-windows)
*   [Why might Siemens imply GFCI breaker will reduce protection on 3-wire dryer circuit?](https://diy.stackexchange.com/questions/257121/why-might-siemens-imply-gfci-breaker-will-reduce-protection-on-3-wire-dryer-circ)
*   [A square number that is a concatenation of at least 25 squares](https://puzzling.stackexchange.com/questions/118075/a-square-number-that-is-a-concatenation-of-at-least-25-squares)
*   [Is it okay that part of the power jack is outside the device?](https://superuser.com/questions/1743565/is-it-okay-that-part-of-the-power-jack-is-outside-the-device)
*   [PhD in a probably hopeless research project](https://academia.stackexchange.com/questions/188928/phd-in-a-probably-hopeless-research-project)
*   [How can I test if an IC output is buffered?](https://electronics.stackexchange.com/questions/635840/how-can-i-test-if-an-ic-output-is-buffered)

[Question feed](https://stackoverflow.com/feeds/question/44399422 "Feed of this question and its answers")