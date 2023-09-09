---
{"title":"java - Jackson databind enum case insensitive - Stack Overflow","url":"https://stackoverflow.com/questions/24157817/jackson-databind-enum-case-insensitive","clipped_at":"2022-07-21 14:46:41","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/java-Jackson-databind-enum-case-insensitive-Stack-Overflow_1658386001/","dgPassFrontmatter":true}
---


# java - Jackson databind enum case insensitive - Stack Overflow

# [Jackson databind enum case insensitive](https://stackoverflow.com/questions/24157817/jackson-databind-enum-case-insensitive)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 8 years, 1 month ago

Modified [1 year, 11 months ago](https://stackoverflow.com/questions/24157817/jackson-databind-enum-case-insensitive?lastactivity "2020-08-24 14:29:15Z")

Viewed 94k times

137

28

[](https://stackoverflow.com/posts/24157817/timeline)

How can I deserialize JSON string that contains enum values that are case insensitive? (using Jackson Databind)

The JSON string:

```java
[{"url": "foo", "type": "json"}]
```

and my Java POJO:

```java
public static class Endpoint {

    public enum DataType {
        JSON, HTML
    }

    public String url;
    public DataType type;

    public Endpoint() {

    }

}
```

in this case,deserializing the JSON with `"type":"json"` would fail where as `"type":"JSON"` would work. But I want `"json"` to work as well for naming convention reasons.

Serializing the POJO also results in upper case `"type":"JSON"`

I thought of using `@JsonCreator` and @JsonGetter:

```java
    @JsonCreator
    private Endpoint(@JsonProperty("name") String url, @JsonProperty("type") String type) {
        this.url = url;
        this.type = DataType.valueOf(type.toUpperCase());
    }

    //....
    @JsonGetter
    private String getType() {
        return type.name().toLowerCase();
    }
```

And it worked. But I was wondering whether there's a better solutuon because this looks like a hack to me.

I can also write a custom deserializer but I got many different POJOs that use enums and it would be hard to maintain.

Can anyone suggest a better way to serialize and deserialize enums with proper naming convention?

**I don't want my enums in java to be lowercase!**

Here is some test code that I used:

```java
    String data = "[{\"url\":\"foo\", \"type\":\"json\"}]";
    Endpoint[] arr = new ObjectMapper().readValue(data, Endpoint[].class);
        System.out.println("POJO[]->" + Arrays.toString(arr));
        System.out.println("JSON ->" + new ObjectMapper().writeValueAsString(arr));
```

[java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [json](https://stackoverflow.com/questions/tagged/json "show questions tagged 'json'") [serialization](https://stackoverflow.com/questions/tagged/serialization "show questions tagged 'serialization'") [enums](https://stackoverflow.com/questions/tagged/enums "show questions tagged 'enums'") [jackson](https://stackoverflow.com/questions/tagged/jackson "show questions tagged 'jackson'")

[Share](https://stackoverflow.com/q/24157817 "Short permalink to this question")

Follow

[edited Apr 8, 2015 at 12:44](https://stackoverflow.com/posts/24157817/revisions "show all edits to this post")

[

![user avatar](/img/user/阅读库藏/assets/1658386001-f8f9b3e7d3e15d41a7a11fe4103822a0.jpg)

](https://stackoverflow.com/users/185034/paul)

[Paul](https://stackoverflow.com/users/185034/paul)

19.2k1313 gold badges7676 silver badges9595 bronze badges

asked Jun 11, 2014 at 8:13

[

![user avatar](/img/user/阅读库藏/assets/1658386001-12a3786cb8c84a161cd70abe27d0134a.jpg)

](https://stackoverflow.com/users/896997/tom91136)

[tom91136](https://stackoverflow.com/users/896997/tom91136)

8,2421212 gold badges5555 silver badges7474 bronze badges

*   Which version of Jackson are you on? Take a look at this JIRA [jira.codehaus.org/browse/JACKSON-861](https://jira.codehaus.org/browse/JACKSON-861)
    
    – [Alexey Gavrilov](https://stackoverflow.com/users/3537858/alexey-gavrilov "10,173 reputation")
    
    [Jun 11, 2014 at 8:22](#comment37282431_24157817)
    
*   I'm using Jackson 2.2.3
    
    – [tom91136](https://stackoverflow.com/users/896997/tom91136 "8,242 reputation")
    
    [Jun 11, 2014 at 8:35](#comment37282809_24157817)
    
*   OK I just updated to 2.4.0-RC3
    
    – [tom91136](https://stackoverflow.com/users/896997/tom91136 "8,242 reputation")
    
    [Jun 11, 2014 at 8:56](#comment37283544_24157817)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 13 Answers

Sorted by:

Trending sort available

Highest score (default) Trending (recent votes count more) Date modified (newest first) Date created (oldest first)

186

[](https://stackoverflow.com/posts/44217670/timeline)

## Jackson 2.9

This is now very simple, using `jackson-databind` 2.9.0 and above

```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.enable(MapperFeature.ACCEPT_CASE_INSENSITIVE_ENUMS);

// objectMapper now deserializes enums in a case-insensitive manner
```

* * *

_Full example with tests_

```java
import com.fasterxml.jackson.databind.MapperFeature;
import com.fasterxml.jackson.databind.ObjectMapper;

public class Main {

  private enum TestEnum { ONE }
  private static class TestObject { public TestEnum testEnum; }

  public static void main (String[] args) {
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.enable(MapperFeature.ACCEPT_CASE_INSENSITIVE_ENUMS);

    try {
      TestObject uppercase = 
        objectMapper.readValue("{ \"testEnum\": \"ONE\" }", TestObject.class);
      TestObject lowercase = 
        objectMapper.readValue("{ \"testEnum\": \"one\" }", TestObject.class);
      TestObject mixedcase = 
        objectMapper.readValue("{ \"testEnum\": \"oNe\" }", TestObject.class);

      if (uppercase.testEnum != TestEnum.ONE) throw new Exception("cannot deserialize uppercase value");
      if (lowercase.testEnum != TestEnum.ONE) throw new Exception("cannot deserialize lowercase value");
      if (mixedcase.testEnum != TestEnum.ONE) throw new Exception("cannot deserialize mixedcase value");

      System.out.println("Success: all deserializations worked");
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
}
```

[Share](https://stackoverflow.com/a/44217670 "Short permalink to this answer")

Follow

[edited Aug 24, 2020 at 14:29](https://stackoverflow.com/posts/44217670/revisions "show all edits to this post")

answered May 27, 2017 at 13:53

[

![user avatar](/img/user/阅读库藏/assets/1658386001-9cf99243208e282c87e771f7a6ce2b88.png)

](https://stackoverflow.com/users/1082449/davnicwil)

[davnicwil](https://stackoverflow.com/users/1082449/davnicwil)

25.2k1414 gold badges9696 silver badges110110 bronze badges

*   5
    
    This one is gold!
    
    – [Vikas Prasad](https://stackoverflow.com/users/1781024/vikas-prasad "2,891 reputation")
    
    [Sep 21, 2017 at 7:14](#comment79634954_44217670)
    
*   10
    
    I'm using 2.9.2 and it doesn't work. Caused by: com.fasterxml.jackson.databind.exc.InvalidFormatException: Cannot deserialize value of type ....Gender\` from String "male": value not one of declared Enum instance names: \[FAMALE, MALE\]
    
    – [Jordan Silva](https://stackoverflow.com/users/1400380/jordan-silva "765 reputation")
    
    [Oct 22, 2017 at 17:19](#comment80700942_44217670)
    
*   @JordanSilva it certainly does work with v2.9.2. I have added a full code example with tests for verification. I don't know what might have happened in your case, but running the example code with `jackson-databind` 2.9.2 specifically works as expected.
    
    – [davnicwil](https://stackoverflow.com/users/1082449/davnicwil "25,201 reputation")
    
    [Feb 6, 2018 at 8:23](#comment84273160_44217670)
    
*   14
    
    using Spring Boot, you can simply add the property `spring.jackson.mapper.accept-case-insensitive-enums=true`
    
    – [Arne Burmeister](https://stackoverflow.com/users/12890/arne-burmeister "19,428 reputation")
    
    [Jun 12, 2019 at 8:06](#comment99695019_44217670)
    
*   1
    
    @JordanSilva maybe you are trying to deserialize enum in get parameters as I did?=) I've solved my problem and answered here. Hope it can help
    
    – [Konstantin Zyubin](https://stackoverflow.com/users/4035262/konstantin-zyubin "650 reputation")
    
    [Sep 4, 2019 at 17:22](#comment102019546_44217670)
    

[Show **1** more comment](# "Expand to show all comments on this post")

103

[](https://stackoverflow.com/posts/24166163/timeline)

I ran into this same issue in my project, we decided to build our enums with a string key and use `@JsonValue` and a static constructor for serialization and deserialization respectively.

```java
public enum DataType {
    JSON("json"), 
    HTML("html");

    private String key;

    DataType(String key) {
        this.key = key;
    }

    @JsonCreator
    public static DataType fromString(String key) {
        return key == null
                ? null
                : DataType.valueOf(key.toUpperCase());
    }

    @JsonValue
    public String getKey() {
        return key;
    }
}
```

[Share](https://stackoverflow.com/a/24166163 "Short permalink to this answer")

Follow

[edited Mar 6, 2015 at 1:26](https://stackoverflow.com/posts/24166163/revisions "show all edits to this post")

answered Jun 11, 2014 at 14:55

[

![user avatar](/img/user/阅读库藏/assets/1658386001-9b34dc7459dd0b335191d7818694f595.jpg)

](https://stackoverflow.com/users/1756430/sam-berry)

[Sam Berry](https://stackoverflow.com/users/1756430/sam-berry)

6,76966 gold badges3636 silver badges5454 bronze badges

*   2
    
    This should be `DataType.valueOf(key.toUpperCase())` - otherwise, you have not really changed anything. Coding defensively to avoid an NPE: `return (null == key ? null : DataType.valueOf(key.toUpperCase()))`
    
    – [sarumont](https://stackoverflow.com/users/43356/sarumont "1,704 reputation")
    
    [Mar 5, 2015 at 20:13](#comment46034917_24166163)
    
*   2
    
    Good catch @sarumont. I have made the edit. Also, renamed method to "fromString" to [play nicely with JAX-RS](http://cxf.apache.org/docs/jax-rs-basics.html#JAX-RSBasics-DealingwithParameters).
    
    – [Sam Berry](https://stackoverflow.com/users/1756430/sam-berry "6,769 reputation")
    
    [Mar 6, 2015 at 1:29](#comment46042803_24166163)
    
*   1
    
    I liked this approach, but went for a less verbose variant, see below.
    
    – [linqu](https://stackoverflow.com/users/1245622/linqu "10,040 reputation")
    
    [Apr 26, 2016 at 10:56](#comment61292932_24166163)
    
*   2
    
    apparently the `key` field is unnecessary. In `getKey`, you could just `return name().toLowerCase()`
    
    – [yair](https://stackoverflow.com/users/978502/yair "8,647 reputation")
    
    [Jun 20, 2016 at 10:39](#comment63291272_24166163)
    
*   1
    
    I like the key field in the case where you want to name the enum something different than what the json will have. In my case a legacy system sends a really abbreviated and hard to remember name for the value it sends, and I can use this field to translate to a better name for my java enum.
    
    – [grinch](https://stackoverflow.com/users/1476154/grinch "774 reputation")
    
    [Jun 8, 2017 at 3:48](#comment75850690_24166163)
    

[Show **5** more comments](# "Expand to show all comments on this post")

66

[](https://stackoverflow.com/posts/37667341/timeline)

Since Jackson 2.6, you can simply do this:

```java
    public enum DataType {
        @JsonProperty("json")
        JSON,
        @JsonProperty("html")
        HTML
    }
```

For a full example, see [this gist](https://gist.github.com/iambic69/b58136a9f513a2fd9459c8753e15d785).

[Share](https://stackoverflow.com/a/37667341 "Short permalink to this answer")

Follow

answered Jun 6, 2016 at 21:43

[

![user avatar](/img/user/阅读库藏/assets/1658386001-b2a1bf0d9ab366fa7a8f63dd2c4d4e57.png)

](https://stackoverflow.com/users/4451384/andrew-bickerton)

[Andrew Bickerton](https://stackoverflow.com/users/4451384/andrew-bickerton)

80966 silver badges22 bronze badges

*   45
    
    Note that doing this will reverse the problem. Now Jackson will only accept lowercase, and reject any uppercase or mixed-case values.
    
    – [Pixel Elephant](https://stackoverflow.com/users/1240320/pixel-elephant "19,363 reputation")
    
    [Jul 26, 2016 at 14:57](#comment64574825_37667341)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

42

[](https://stackoverflow.com/posts/24173645/timeline)

In version 2.4.0 you can register a custom serializer for all the Enum types ([link](https://github.com/FasterXML/jackson-databind/issues/227) to the github issue). Also you can replace the standard Enum deserializer on your own that will be aware about the Enum type. Here is an example:

```java
public class JacksonEnum {

    public static enum DataType {
        JSON, HTML
    }

    public static void main(String[] args) throws IOException {
        List<DataType> types = Arrays.asList(JSON, HTML);
        ObjectMapper mapper = new ObjectMapper();
        SimpleModule module = new SimpleModule();
        module.setDeserializerModifier(new BeanDeserializerModifier() {
            @Override
            public JsonDeserializer<Enum> modifyEnumDeserializer(DeserializationConfig config,
                                                              final JavaType type,
                                                              BeanDescription beanDesc,
                                                              final JsonDeserializer<?> deserializer) {
                return new JsonDeserializer<Enum>() {
                    @Override
                    public Enum deserialize(JsonParser jp, DeserializationContext ctxt) throws IOException {
                        Class<? extends Enum> rawClass = (Class<Enum<?>>) type.getRawClass();
                        return Enum.valueOf(rawClass, jp.getValueAsString().toUpperCase());
                    }
                };
            }
        });
        module.addSerializer(Enum.class, new StdSerializer<Enum>(Enum.class) {
            @Override
            public void serialize(Enum value, JsonGenerator jgen, SerializerProvider provider) throws IOException {
                jgen.writeString(value.name().toLowerCase());
            }
        });
        mapper.registerModule(module);
        String json = mapper.writeValueAsString(types);
        System.out.println(json);
        List<DataType> types2 = mapper.readValue(json, new TypeReference<List<DataType>>() {});
        System.out.println(types2);
    }
}
```

Output:

```java
["json","html"]
[JSON, HTML]
```

[Share](https://stackoverflow.com/a/24173645 "Short permalink to this answer")

Follow

answered Jun 11, 2014 at 22:27

[

![user avatar](/img/user/阅读库藏/assets/1658386001-8830808b8b3703c53d2617e35d0bf594.jpg)

](https://stackoverflow.com/users/3537858/alexey-gavrilov)

[Alexey Gavrilov](https://stackoverflow.com/users/3537858/alexey-gavrilov)

10.2k22 gold badges3434 silver badges4545 bronze badges

*   1
    
    Thanks, now I can remove all the boilerplate in my POJO :)
    
    – [tom91136](https://stackoverflow.com/users/896997/tom91136 "8,242 reputation")
    
    [Jun 13, 2014 at 8:10](#comment37363035_24173645)
    
*   I personally am advocating for this in my projects. If you look at my example, it requires a lot of boilerplate code. One benefit of using a separate attributes for de/serialization is that it decouples the names of the Java-important values (enum names) to client-important values (pretty print). E.g., if it was desired to change the HTML DataType to HTML\_DATA\_TYPE, you could do so without affecting the external API, if there is a key specified.
    
    – [Sam Berry](https://stackoverflow.com/users/1756430/sam-berry "6,769 reputation")
    
    [Mar 9, 2015 at 7:24](#comment46128918_24173645)
    
*   1
    
    This is a good start but it will fail if your enum is using JsonProperty or JsonCreator. Dropwizard has [FuzzyEnumModule](https://github.com/dropwizard/dropwizard/blob/master/dropwizard-jackson/src/main/java/io/dropwizard/jackson/FuzzyEnumModule.java) which is a more robust implementation.
    
    – [Pixel Elephant](https://stackoverflow.com/users/1240320/pixel-elephant "19,363 reputation")
    
    [Feb 22, 2017 at 20:10](#comment71949526_24173645)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

35

[](https://stackoverflow.com/posts/54692478/timeline)

If you're using Spring Boot `2.1.x` with Jackson `2.9` you can simply use this application property:

`spring.jackson.mapper.accept-case-insensitive-enums=true`

[Share](https://stackoverflow.com/a/54692478 "Short permalink to this answer")

Follow

[edited Feb 14, 2019 at 14:43](https://stackoverflow.com/posts/54692478/revisions "show all edits to this post")

answered Feb 14, 2019 at 14:14

[

![user avatar](/img/user/阅读库藏/assets/1658386001-73a2ea67e7a40ace94dae107eaceae21.png)

](https://stackoverflow.com/users/5618041/harpresing)

[harpresing](https://stackoverflow.com/users/5618041/harpresing)

54344 silver badges1010 bronze badges

*   See documentation on [Customize Jackson from Spring boot](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#howto-customize-the-jackson-objectmapper). and list of customization point for mapper as Enum in Jackson documentation on [com.fasterxml.jackson.databind.MapperFeature API doc](https://fasterxml.github.io/jackson-databind/javadoc/2.9/)
    
    – [Ahmad Hoghooghi](https://stackoverflow.com/users/2246238/ahmad-hoghooghi "135 reputation")
    
    [Feb 20, 2021 at 10:34](#comment117196900_54692478)
    
*   1
    
    Running Spring Boot 2.4.5 and Jackson 2.11 - doesn't work
    
    – [Geyser14](https://stackoverflow.com/users/2735802/geyser14 "1,225 reputation")
    
    [Sep 27, 2021 at 6:49](#comment122560306_54692478)
    
*   Running Spring Boot 2.5.5 works !!!
    
    – [James Dube](https://stackoverflow.com/users/7243065/james-dube "608 reputation")
    
    [Feb 23 at 14:12](#comment125921920_54692478)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

34

[](https://stackoverflow.com/posts/36862870/timeline)

I went for the solution of _Sam B._ but a simpler variant.

```java
public enum Type {
    PIZZA, APPLE, PEAR, SOUP;

    @JsonCreator
    public static Type fromString(String key) {
        for(Type type : Type.values()) {
            if(type.name().equalsIgnoreCase(key)) {
                return type;
            }
        }
        return null;
    }
}
```

[Share](https://stackoverflow.com/a/36862870 "Short permalink to this answer")

Follow

[edited Apr 26, 2016 at 14:26](https://stackoverflow.com/posts/36862870/revisions "show all edits to this post")

answered Apr 26, 2016 at 10:55

[

![user avatar](/img/user/阅读库藏/assets/1658386001-0bb73c3b937e499c03edafc4fc68bbe0.jpg)

](https://stackoverflow.com/users/1245622/linqu)

[linqu](https://stackoverflow.com/users/1245622/linqu)

10k77 gold badges5252 silver badges6363 bronze badges

*   I don't think this is simpler. `DataType.valueOf(key.toUpperCase())` is a direct instantiation where you have a loop. This could be an issue for a very numerous enum. Of course, `valueOf` can throw an IllegalArgumentException, which your code avoids, so that's a good benefit if you prefer null checking to exception checking.
    
    – [Patrick M](https://stackoverflow.com/users/1146608/patrick-m "10,087 reputation")
    
    [Dec 13, 2019 at 15:40](#comment104850661_36862870)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

9

[](https://stackoverflow.com/posts/57793147/timeline)

For those who tries to deserialize Enum ignoring case in **GET parameters**, enabling ACCEPT\_CASE\_INSENSITIVE\_ENUMS will not do any good. It won't help because this option only works for **body deserialization**. Instead try this:

```java
public class StringToEnumConverter implements Converter<String, Modes> {
    @Override
    public Modes convert(String from) {
        return Modes.valueOf(from.toUpperCase());
    }
}
```

and then

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addFormatters(FormatterRegistry registry) {
        registry.addConverter(new StringToEnumConverter());
    }
}
```

The answer and code samples are from [here](https://www.baeldung.com/spring-mvc-custom-data-binder)

[Share](https://stackoverflow.com/a/57793147 "Short permalink to this answer")

Follow

[edited Sep 4, 2019 at 17:27](https://stackoverflow.com/posts/57793147/revisions "show all edits to this post")

answered Sep 4, 2019 at 17:20

[

![user avatar](/img/user/阅读库藏/assets/1658386001-759083bc7251185420534bd8e244d77e.jpg)

](https://stackoverflow.com/users/4035262/konstantin-zyubin)

[Konstantin Zyubin](https://stackoverflow.com/users/4035262/konstantin-zyubin)

6501111 silver badges1616 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

5

[](https://stackoverflow.com/posts/61737600/timeline)

To allow case insensitive deserialization of enums in jackson, simply add the below property to the `application.properties` file of your spring boot project.

```java
spring.jackson.mapper.accept-case-insensitive-enums=true
```

If you have the yaml version of properties file, add below property to your `application.yml` file.

```java
spring:
  jackson:
    mapper:
      accept-case-insensitive-enums: true
```

[Share](https://stackoverflow.com/a/61737600 "Short permalink to this answer")

Follow

answered May 11, 2020 at 19:18

[

![user avatar](/img/user/阅读库藏/assets/1658386001-a7c3bcb234980503be0b1b610cb03800.png)

](https://stackoverflow.com/users/814548/vivek)

[Vivek](https://stackoverflow.com/users/814548/vivek)

10.3k1919 gold badges8181 silver badges113113 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

1

[](https://stackoverflow.com/posts/60251159/timeline)

With apologies to @Konstantin Zyubin, his answer was close to what I needed - but I didn't understand it, so here's how I think it should go:

If you want to deserialize one enum type as case insensitive - i.e. you don't want to, or can't, modify the behavior of the entire application, you can create a custom deserializer just for one type - by sub-classing `StdConverter` and force Jackson to use it only on the relevant fields using the `JsonDeserialize` annotation.

Example:

```java
public class ColorHolder {

  public enum Color {
    RED, GREEN, BLUE
  }

  public static final class ColorParser extends StdConverter<String, Color> {
    @Override
    public Color convert(String value) {
      return Arrays.stream(Color.values())
        .filter(e -> e.getName().equalsIgnoreCase(value.trim()))
        .findFirst()
        .orElseThrow(() -> new IllegalArgumentException("Invalid value '" + value + "'"));
    }
  }

  @JsonDeserialize(converter = ColorParser.class)
  Color color;
}
```

[Share](https://stackoverflow.com/a/60251159 "Short permalink to this answer")

Follow

answered Feb 16, 2020 at 17:25

[

![user avatar](/img/user/阅读库藏/assets/1658386001-ab412f11212fe06c26d776deaae8dbf9.png)

](https://stackoverflow.com/users/53538/guss)

[Guss](https://stackoverflow.com/users/53538/guss)

27.7k1616 gold badges9696 silver badges122122 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/26036465/timeline)

Problem is releated to [com.fasterxml.jackson.databind.util.EnumResolver](https://github.com/FasterXML/jackson-databind/blob/master/src/main/java/com/fasterxml/jackson/databind/util/EnumResolver.java#L120). it uses HashMap to hold enum values and HashMap doesn't support case insensitive keys.

in answers above, all chars should be uppercase or lowercase. but I fixed all (in)sensitive problems for enums with that:

[https://gist.github.com/bhdrk/02307ba8066d26fa1537](https://gist.github.com/bhdrk/02307ba8066d26fa1537)

**CustomDeserializers.java**

```java
import com.fasterxml.jackson.databind.BeanDescription;
import com.fasterxml.jackson.databind.DeserializationConfig;
import com.fasterxml.jackson.databind.JsonDeserializer;
import com.fasterxml.jackson.databind.JsonMappingException;
import com.fasterxml.jackson.databind.deser.std.EnumDeserializer;
import com.fasterxml.jackson.databind.module.SimpleDeserializers;
import com.fasterxml.jackson.databind.util.EnumResolver;

import java.util.HashMap;
import java.util.Map;


public class CustomDeserializers extends SimpleDeserializers {

    @Override
    @SuppressWarnings("unchecked")
    public JsonDeserializer<?> findEnumDeserializer(Class<?> type, DeserializationConfig config, BeanDescription beanDesc) throws JsonMappingException {
        return createDeserializer((Class<Enum>) type);
    }

    private <T extends Enum<T>> JsonDeserializer<?> createDeserializer(Class<T> enumCls) {
        T[] enumValues = enumCls.getEnumConstants();
        HashMap<String, T> map = createEnumValuesMap(enumValues);
        return new EnumDeserializer(new EnumCaseInsensitiveResolver<T>(enumCls, enumValues, map));
    }

    private <T extends Enum<T>> HashMap<String, T> createEnumValuesMap(T[] enumValues) {
        HashMap<String, T> map = new HashMap<String, T>();
        // from last to first, so that in case of duplicate values, first wins
        for (int i = enumValues.length; --i >= 0; ) {
            T e = enumValues[i];
            map.put(e.toString(), e);
        }
        return map;
    }

    public static class EnumCaseInsensitiveResolver<T extends Enum<T>> extends EnumResolver<T> {
        protected EnumCaseInsensitiveResolver(Class<T> enumClass, T[] enums, HashMap<String, T> map) {
            super(enumClass, enums, map);
        }

        @Override
        public T findEnum(String key) {
            for (Map.Entry<String, T> entry : _enumsById.entrySet()) {
                if (entry.getKey().equalsIgnoreCase(key)) { // magic line <--
                    return entry.getValue();
                }
            }
            return null;
        }
    }
}
```

**Usage:**

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.module.SimpleModule;


public class JSON {

    public static void main(String[] args) {
        SimpleModule enumModule = new SimpleModule();
        enumModule.setDeserializers(new CustomDeserializers());

        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(enumModule);
    }

}
```

[Share](https://stackoverflow.com/a/26036465 "Short permalink to this answer")

Follow

answered Sep 25, 2014 at 10:37

[

![user avatar](/img/user/阅读库藏/assets/1658386001-5daa86c27beab20c82bfd36a25307940.png)

](https://stackoverflow.com/users/586522/bhdrk)

[bhdrk](https://stackoverflow.com/users/586522/bhdrk)

3,1792424 silver badges1818 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/53092208/timeline)

I used a modification of Iago Fernández and Paul solution .

I had an enum in my requestobject which needed to be case insensitive

```java
@POST
public Response doSomePostAction(RequestObject object){
 //resource implementation
}



class RequestObject{
 //other params 
 MyEnumType myType;

 @JsonSetter
 public void setMyType(String type){
   myType = MyEnumType.valueOf(type.toUpperCase());
 }
 @JsonGetter
 public String getType(){
   return myType.toString();//this can change 
 }
}
```

[Share](https://stackoverflow.com/a/53092208 "Short permalink to this answer")

Follow

[edited Jan 7, 2020 at 12:59](https://stackoverflow.com/posts/53092208/revisions "show all edits to this post")

[

![user avatar](/img/user/阅读库藏/assets/1658386001-bce878d0318dcbc306a18abf63ce5eee.png)

](https://stackoverflow.com/users/-1/community)

[Community](https://stackoverflow.com/users/-1/community)Bot

111 silver badge

answered Oct 31, 2018 at 21:29

[

![user avatar](/img/user/阅读库藏/assets/1658386001-ed65e8f57eb51ad7b348763aa7c364cc.jpg)

](https://stackoverflow.com/users/3122385/trooper31)

[trooper31](https://stackoverflow.com/users/3122385/trooper31)

18211 gold badge22 silver badges88 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

\-1

[](https://stackoverflow.com/posts/29515266/timeline)

Here's how I sometimes handle enums when I want to deserialize in a case-insensitive manner (building on the code posted in the question):

```java
@JsonIgnore
public void setDataType(DataType dataType)
{
  type = dataType;
}

@JsonProperty
public void setDataType(String dataType)
{
  // Clean up/validate String however you want. I like
  // org.apache.commons.lang3.StringUtils.trimToEmpty
  String d = StringUtils.trimToEmpty(dataType).toUpperCase();
  setDataType(DataType.valueOf(d));
}
```

If the enum is non-trivial and thus in its own class I usually add a static parse method to handle lowercase Strings.

[Share](https://stackoverflow.com/a/29515266 "Short permalink to this answer")

Follow

answered Apr 8, 2015 at 12:55

[

![user avatar](/img/user/阅读库藏/assets/1658386001-f8f9b3e7d3e15d41a7a11fe4103822a0.jpg)

](https://stackoverflow.com/users/185034/paul)

[Paul](https://stackoverflow.com/users/185034/paul)

19.2k1313 gold badges7676 silver badges9595 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

\-1

[](https://stackoverflow.com/posts/32802851/timeline)

Deserialize enum with jackson is simple. When you want deserialize enum based in String need a constructor, a getter and a setter to your enum.Also class that use that enum must have a setter which receive DataType as param, not String:

```java
public class Endpoint {

     public enum DataType {
        JSON("json"), HTML("html");

        private String type;

        @JsonValue
        public String getDataType(){
           return type;
        }

        @JsonSetter
        public void setDataType(String t){
           type = t.toLowerCase();
        }
     }

     public String url;
     public DataType type;

     public Endpoint() {

     }

     public void setType(DataType dataType){
        type = dataType;
     }

}
```

When you have your json, you can deserialize to Endpoint class using ObjectMapper of Jackson:

```java
ObjectMapper mapper = new ObjectMapper();
mapper.enable(SerializationFeature.INDENT_OUTPUT);
try {
    Endpoint endpoint = mapper.readValue("{\"url\":\"foo\",\"type\":\"json\"}", Endpoint.class);
} catch (IOException e1) {
        // TODO Auto-generated catch block
    e1.printStackTrace();
}
```

[Share](https://stackoverflow.com/a/32802851 "Short permalink to this answer")

Follow

[edited Sep 27, 2015 at 16:02](https://stackoverflow.com/posts/32802851/revisions "show all edits to this post")

answered Sep 26, 2015 at 22:57

[

![user avatar](/img/user/阅读库藏/assets/1658386001-78b5ea0bb4459e653a8c02932aad10ef.jpg)

](https://stackoverflow.com/users/5342071/iago-fern%c3%a1ndez)

[Iago Fernández](https://stackoverflow.com/users/5342071/iago-fern%c3%a1ndez)

1133 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

  

## Your Answer

### Sign up or [log in](https://stackoverflow.com/users/login?ssrc=question_page&returnurl=https%3a%2f%2fstackoverflow.com%2fquestions%2f24157817%2fjackson-databind-enum-case-insensitive%23new-answer)

Sign up using Google

Sign up using Facebook

Sign up using Email and Password

 

### Post as a guest

Name

Email

Required, but never shown

Post Your Answer

By clicking “Post Your Answer”, you agree to our [terms of service](https://stackoverflow.com/legal/terms-of-service/public), [privacy policy](https://stackoverflow.com/legal/privacy-policy) and [cookie policy](https://stackoverflow.com/legal/cookie-policy)

## Not the answer you're looking for? Browse other questions tagged [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [json](https://stackoverflow.com/questions/tagged/json "show questions tagged 'json'") [serialization](https://stackoverflow.com/questions/tagged/serialization "show questions tagged 'serialization'") [enums](https://stackoverflow.com/questions/tagged/enums "show questions tagged 'enums'") [jackson](https://stackoverflow.com/questions/tagged/jackson "show questions tagged 'jackson'") or [ask your own question](https://stackoverflow.com/questions/ask).

The Overflow Blog

*   [Code completion isn’t magic; it just feels that way (Ep. 464)](https://stackoverflow.blog/2022/07/19/code-completion-isnt-magic-it-just-feels-that-way-ep-464/?cb=1)
    
*   [How APIs can take the pain out of legacy system headaches (Ep. 465)](https://stackoverflow.blog/2022/07/20/opentext-api-legacy-systems-podcast-ep-465/?cb=1)
    

Featured on Meta

*   [Announcing the Stacks Editor Beta release!](https://meta.stackexchange.com/questions/380295/announcing-the-stacks-editor-beta-release?cb=1)
    
*   [Trending: A new answer sorting option](https://meta.stackoverflow.com/questions/418767/trending-a-new-answer-sorting-option?cb=1)
    
*   [The \[options\] tag is being burninated](https://meta.stackoverflow.com/questions/417320/the-options-tag-is-being-burninated?cb=1)
    

#### Linked

[

2

](https://stackoverflow.com/q/67741152?lq=1 "Question score (upvotes - downvotes)")[Make Enums case insensitive](https://stackoverflow.com/questions/67741152/make-enums-case-insensitive?noredirect=1&lq=1)

[

18

](https://stackoverflow.com/q/50231233?lq=1 "Question score (upvotes - downvotes)")[Deserialize enum ignoring case in Spring Boot controller](https://stackoverflow.com/questions/50231233/deserialize-enum-ignoring-case-in-spring-boot-controller?noredirect=1&lq=1)

[

7

](https://stackoverflow.com/q/49175511?lq=1 "Question score (upvotes - downvotes)")[How allow case-insensitive mapping of enums in jackson/Spring boot?](https://stackoverflow.com/questions/49175511/how-allow-case-insensitive-mapping-of-enums-in-jackson-spring-boot?noredirect=1&lq=1)

[

4

](https://stackoverflow.com/q/31325147?lq=1 "Question score (upvotes - downvotes)")[Unrecognised Property with Jackson @JsonCreator](https://stackoverflow.com/questions/31325147/unrecognised-property-with-jackson-jsoncreator?noredirect=1&lq=1)

[

3

](https://stackoverflow.com/q/25079332?lq=1 "Question score (upvotes - downvotes)")[Parse Enum by value using SnakeYAML](https://stackoverflow.com/questions/25079332/parse-enum-by-value-using-snakeyaml?noredirect=1&lq=1)

[

2

](https://stackoverflow.com/q/62738568?lq=1 "Question score (upvotes - downvotes)")[custom xml deserializer in jackson for enum](https://stackoverflow.com/questions/62738568/custom-xml-deserializer-in-jackson-for-enum?noredirect=1&lq=1)

[

1

](https://stackoverflow.com/q/71527893?lq=1 "Question score (upvotes - downvotes)")[How to make enum parameter lowercase in SpringBoot/Swagger?](https://stackoverflow.com/questions/71527893/how-to-make-enum-parameter-lowercase-in-springboot-swagger?noredirect=1&lq=1)

[

\-1

](https://stackoverflow.com/q/49999748?lq=1 "Question score (upvotes - downvotes)")[Enum, How to rename during (de)serialization?](https://stackoverflow.com/questions/49999748/enum-how-to-rename-during-deserialization?noredirect=1&lq=1)

[

1

](https://stackoverflow.com/q/44446110?lq=1 "Question score (upvotes - downvotes)")[Bind a String value to an enum in a @RequestBody entity in Spring Boot](https://stackoverflow.com/questions/44446110/bind-a-string-value-to-an-enum-in-a-requestbody-entity-in-spring-boot?noredirect=1&lq=1)

#### Related

[

1649

](https://stackoverflow.com/q/8447?rq=1 "Question score (upvotes - downvotes)")[What does the \[Flags\] Enum Attribute mean in C#?](https://stackoverflow.com/questions/8447/what-does-the-flags-enum-attribute-mean-in-c?rq=1)

[

3696

](https://stackoverflow.com/q/29482?rq=1 "Question score (upvotes - downvotes)")[How do I cast int to enum in C#?](https://stackoverflow.com/questions/29482/how-do-i-cast-int-to-enum-in-c?rq=1)

[

4210

](https://stackoverflow.com/q/105372?rq=1 "Question score (upvotes - downvotes)")[How to enumerate an enum](https://stackoverflow.com/questions/105372/how-to-enumerate-an-enum?rq=1)

[

2265

](https://stackoverflow.com/q/604424?rq=1 "Question score (upvotes - downvotes)")[How to get an enum value from a string value in Java](https://stackoverflow.com/questions/604424/how-to-get-an-enum-value-from-a-string-value-in-java?rq=1)

[

2198

](https://stackoverflow.com/q/943398?rq=1 "Question score (upvotes - downvotes)")[Get int value from enum in C#](https://stackoverflow.com/questions/943398/get-int-value-from-enum-in-c-sharp?rq=1)

[

2021

](https://stackoverflow.com/q/1750435?rq=1 "Question score (upvotes - downvotes)")[Comparing Java enum members: == or equals()?](https://stackoverflow.com/questions/1750435/comparing-java-enum-members-or-equals?rq=1)

[

1335

](https://stackoverflow.com/q/2441290?rq=1 "Question score (upvotes - downvotes)")[JavaScriptSerializer - JSON serialization of enum as string](https://stackoverflow.com/questions/2441290/javascriptserializer-json-serialization-of-enum-as-string?rq=1)

[

877

](https://stackoverflow.com/q/4486787?rq=1 "Question score (upvotes - downvotes)")[Jackson with JSON: Unrecognized field, not marked as ignorable](https://stackoverflow.com/questions/4486787/jackson-with-json-unrecognized-field-not-marked-as-ignorable?rq=1)

[

988

](https://stackoverflow.com/q/6349421?rq=1 "Question score (upvotes - downvotes)")[How to use Jackson to deserialise an array of objects](https://stackoverflow.com/questions/6349421/how-to-use-jackson-to-deserialise-an-array-of-objects?rq=1)

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [What purpose are these openings on the roof?](https://diy.stackexchange.com/questions/253274/what-purpose-are-these-openings-on-the-roof)
*   [How do I replace a toilet supply stop valve attached to copper pipe?](https://diy.stackexchange.com/questions/253231/how-do-i-replace-a-toilet-supply-stop-valve-attached-to-copper-pipe)
*   [mv fails with "No space left on device" when the destination has 31 GB of space remaining](https://askubuntu.com/questions/1419651/mv-fails-with-no-space-left-on-device-when-the-destination-has-31-gb-of-space)
*   [Blondie's Heart of Glass shimmering cascade effect](https://music.stackexchange.com/questions/123915/blondies-heart-of-glass-shimmering-cascade-effect)
*   [Involution map, and induced morphism in K-theory](https://mathoverflow.net/questions/426998/involution-map-and-induced-morphism-in-k-theory)
*   [Forgot to pass through customs](https://travel.stackexchange.com/questions/175039/forgot-to-pass-through-customs)
*   [Why didn't GLaDOS kill Chell?](https://gaming.stackexchange.com/questions/398243/why-didnt-glados-kill-chell)
*   [Minimizing under argmin constraint](https://or.stackexchange.com/questions/8714/minimizing-under-argmin-constraint)
*   [Presidential line of succession and age](https://law.stackexchange.com/questions/82299/presidential-line-of-succession-and-age)
*   [Are shrivelled chilis safe to eat and process into chili flakes?](https://cooking.stackexchange.com/questions/121095/are-shrivelled-chilis-safe-to-eat-and-process-into-chili-flakes)
*   [What's inside the SPIKE Essential small angular motor?](https://bricks.stackexchange.com/questions/17305/whats-inside-the-spike-essential-small-angular-motor)
*   [there was/were a number of](https://ell.stackexchange.com/questions/319276/there-was-were-a-number-of)
*   [How to avoid paradoxes about time-ordering operation?](https://physics.stackexchange.com/questions/719187/how-to-avoid-paradoxes-about-time-ordering-operation)
*   [Short satire about a comically upscaled spaceship](https://scifi.stackexchange.com/questions/265695/short-satire-about-a-comically-upscaled-spaceship)
*   [Was 36-bits needed for LISP?](https://retrocomputing.stackexchange.com/questions/24867/was-36-bits-needed-for-lisp)
*   [Is the fact that ZFC implies that 1+1=2 an absolute truth?](https://philosophy.stackexchange.com/questions/92335/is-the-fact-that-zfc-implies-that-11-2-an-absolute-truth)
*   [What are the "disks" seen on the walls of some NASA space shuttles?](https://space.stackexchange.com/questions/59784/what-are-the-disks-seen-on-the-walls-of-some-nasa-space-shuttles)
*   [Looking for a middle ground between raw random and shuffle bags](https://gamedev.stackexchange.com/questions/201805/looking-for-a-middle-ground-between-raw-random-and-shuffle-bags)
*   [In the US, how do we make tax withholding less if we lost our job for a few months?](https://money.stackexchange.com/questions/151838/in-the-us-how-do-we-make-tax-withholding-less-if-we-lost-our-job-for-a-few-mont)
*   [Is "Occupation Japan" idiomatic? (instead of occupation of Japan, occupied Japan or Occupation-era Japan)](https://ell.stackexchange.com/questions/319220/is-occupation-japan-idiomatic-instead-of-occupation-of-japan-occupied-japan)
*   [Is this video of a fast-moving river of lava authentic?](https://skeptics.stackexchange.com/questions/53562/is-this-video-of-a-fast-moving-river-of-lava-authentic)
*   [How to incorporate Lifestyle into D&D?](https://rpg.stackexchange.com/questions/200052/how-to-incorporate-lifestyle-into-dd)
*   [How do the electrical characteristics of an ADC degrade over lifetime?](https://electronics.stackexchange.com/questions/628156/how-do-the-electrical-characteristics-of-an-adc-degrade-over-lifetime)
*   [Are there provisions for a tie in the Conservative leadership election?](https://politics.stackexchange.com/questions/74323/are-there-provisions-for-a-tie-in-the-conservative-leadership-election)

[Question feed](https://stackoverflow.com/feeds/question/24157817 "Feed of this question and its answers")