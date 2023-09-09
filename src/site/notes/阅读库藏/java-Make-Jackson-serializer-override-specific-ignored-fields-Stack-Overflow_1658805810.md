---
{"title":"java - Make Jackson serializer override specific ignored fields - Stack Overflow","url":"https://stackoverflow.com/questions/58763305/make-jackson-serializer-override-specific-ignored-fields","clipped_at":"2022-07-26 11:23:30","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/java-Make-Jackson-serializer-override-specific-ignored-fields-Stack-Overflow_1658805810/","dgPassFrontmatter":true}
---


# java - Make Jackson serializer override specific ignored fields - Stack Overflow

# [Make Jackson serializer override specific ignored fields](https://stackoverflow.com/questions/58763305/make-jackson-serializer-override-specific-ignored-fields)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 2 years, 8 months ago

Modified [2 years, 8 months ago](https://stackoverflow.com/questions/58763305/make-jackson-serializer-override-specific-ignored-fields?lastactivity "2019-11-08 10:07:56Z")

Viewed 893 times

2

[](https://stackoverflow.com/posts/58763305/timeline)

I have Jackson annotated class like this :

```java
public class MyClass {
   String field1;

   @JsonIgnore
   String field2;

   String field3;

   @JsonIgnore
   String field4;
}
```

Assume that I cannot change MyClass code. Then, how can I make ObjectMapper override the JsonIgnore for field2 only and serialize it to json ? I want it to ignore field4 though. Is this easy and few lines of code ?

My code for regular serialization :

```java
public String toJson(SomeObject obj){
    ObjectWriter ow = new ObjectMapper().writer().withDefaultPrettyPrinter();
    String json = null;
    try {
        json = ow.writeValueAsString(obj);
    } catch (JsonProcessingException e) {
        e.printStackTrace();
    }
    return json;
}
```

[java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [json](https://stackoverflow.com/questions/tagged/json "show questions tagged 'json'") [jackson](https://stackoverflow.com/questions/tagged/jackson "show questions tagged 'jackson'")

[Share](https://stackoverflow.com/q/58763305 "Short permalink to this question")

[Improve this question](https://stackoverflow.com/posts/58763305/edit)

Follow

asked Nov 8, 2019 at 8:55

[

![user avatar](/img/user/阅读库藏/assets/1658805810-27cee0e35f2deb77de88d1dc807bbd13.jpg)

](https://stackoverflow.com/users/3184475/erran-morad)

[Erran Morad](https://stackoverflow.com/users/3184475/erran-morad)

4,3431010 gold badges4040 silver badges6969 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 2 Answers

Sorted by:

Trending sort available

Highest score (default) Trending (recent votes count more) Date modified (newest first) Date created (oldest first)

5

[](https://stackoverflow.com/posts/58764437/timeline)

You can use `MixIn` feature:

```java
import com.fasterxml.jackson.annotation.JsonIgnore;
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;

public class JsonApp {

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        mapper.enable(SerializationFeature.INDENT_OUTPUT);
        mapper.addMixIn(MyClass.class, MyClassMixIn.class);

        System.out.println(mapper.writeValueAsString(new MyClass()));
    }
}

interface MyClassMixIn {

    @JsonProperty
    String getField2();
}
```

Above code prints:

```java
{
  "field1" : "F1",
  "field2" : "F2",
  "field3" : "F3"
}
```

[Share](https://stackoverflow.com/a/58764437 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/58764437/edit)

Follow

answered Nov 8, 2019 at 10:07

[

![user avatar](/img/user/阅读库藏/assets/1658805810-58835481d876c96926b936544fa69c9f.jpg)

](https://stackoverflow.com/users/51591/micha%c5%82-ziober)

[Michał Ziober](https://stackoverflow.com/users/51591/micha%c5%82-ziober)

34.7k1717 gold badges9090 silver badges135135 bronze badges

*   Thanks. In the mixin, how can we specify the field name instead of the getter ?
    
    – [Erran Morad](https://stackoverflow.com/users/3184475/erran-morad "4,343 reputation")
    
    [Nov 10, 2019 at 3:09](#comment103853860_58764437)
    
*   1
    
    @BoratSagdiyev, try `@JsonProperty("anotherName")`
    
    – [Michał Ziober](https://stackoverflow.com/users/51591/micha%c5%82-ziober "34,680 reputation")
    
    [Nov 10, 2019 at 11:10](#comment103858178_58764437)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

1

[](https://stackoverflow.com/posts/58763785/timeline)

You could use a custom serializer, like so (I leave the refactoring up to you):

```java
private String getJson(final MyClass obj) {
    final ObjectMapper om = new ObjectMapper();
    om.registerModule(new SimpleModule(
            "CustomModule",
            Version.unknownVersion(),
            Collections.emptyMap(),
            Collections.singletonList(new StdSerializer<MyClass>(MyClass.class) {

                @Override
                public void serialize(final MyClass value, final JsonGenerator jgen, final SerializerProvider provider) throws IOException {
                    jgen.writeStartObject();
                    jgen.writeStringField("field1", value.field1);
                    jgen.writeStringField("field2", value.field2);
                    jgen.writeStringField("field3", value.field3);
                    jgen.writeEndObject();
                }

            })));

    try {
        return om.writeValueAsString(obj);
    } catch (final JsonProcessingException e) {
        throw new RuntimeException(e);
    }
}
```

You can find more information about custom serializers on google/stackoverflow, I think I answered something similar [here](https://stackoverflow.com/questions/58076716/custom-annotation-based-ignore-field-from-serialisation/58077671#58077671)

[Share](https://stackoverflow.com/a/58763785 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/58763785/edit)

Follow

answered Nov 8, 2019 at 9:27

[

![user avatar](/img/user/阅读库藏/assets/1658805810-ce2c36b8780e5e766843eaa3053cab66.png)

](https://stackoverflow.com/users/6473398/sfiss)

[sfiss](https://stackoverflow.com/users/6473398/sfiss)

1,8191111 silver badges1919 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

  

## Your Answer

### Sign up or [log in](https://stackoverflow.com/users/login?ssrc=question_page&returnurl=https%3a%2f%2fstackoverflow.com%2fquestions%2f58763305%2fmake-jackson-serializer-override-specific-ignored-fields%23new-answer)

Sign up using Google

Sign up using Facebook

Sign up using Email and Password

 

### Post as a guest

Name

Email

Required, but never shown

Post Your Answer

By clicking “Post Your Answer”, you agree to our [terms of service](https://stackoverflow.com/legal/terms-of-service/public), [privacy policy](https://stackoverflow.com/legal/privacy-policy) and [cookie policy](https://stackoverflow.com/legal/cookie-policy)

## Not the answer you're looking for? Browse other questions tagged [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [json](https://stackoverflow.com/questions/tagged/json "show questions tagged 'json'") [jackson](https://stackoverflow.com/questions/tagged/jackson "show questions tagged 'jackson'") or [ask your own question](https://stackoverflow.com/questions/ask).

The Overflow Blog

*   [Game Boy emulators, PowerPoint developers, and the enduring appeal of Pokémon...](https://stackoverflow.blog/2022/07/22/game-boy-emulators-powerpoint-developers-and-the-enduring-appeal-of-pokemon-go-ep-466/?cb=1 "Game Boy emulators, PowerPoint developers, and the enduring appeal of Pokémon GO (Ep. 466)")
    
*   [Automate the boring parts of your job](https://stackoverflow.blog/2022/07/25/automate-the-boring-parts-of-your-job/?cb=1)
    

Featured on Meta

*   [Announcing the Stacks Editor Beta release!](https://meta.stackexchange.com/questions/380295/announcing-the-stacks-editor-beta-release?cb=1)
    
*   [Trending: A new answer sorting option](https://meta.stackoverflow.com/questions/418767/trending-a-new-answer-sorting-option?cb=1)
    
*   [The \[options\] tag is being burninated](https://meta.stackoverflow.com/questions/417320/the-options-tag-is-being-burninated?cb=1)
    

[22 people chatting](https://chat.stackoverflow.com/ "22 users active in 22 rooms the last 60 minutes")

[Java](https://chat.stackoverflow.com//rooms/139)

3 hours ago - [OakBot](https://chat.stackoverflow.com//users/4258326)

[![](assets/1658805810-7fc8a6589d7aca75de5aa61843881fe1.jpg "OakBot: 3 hours ago")](https://chat.stackoverflow.com//users/4258326)[![](assets/1658805810-9422acead23e895e4a2e9248f30b2173.png "Johannes Kuhn: Jun 29 at 10:15")](https://chat.stackoverflow.com//users/845414)[![](assets/1658805810-2ca9432d1acf123cfb2fb1820faa15f6.png "jay m")](https://chat.stackoverflow.com//users/19082517)

[Android](https://chat.stackoverflow.com//rooms/15)

7 hours ago - [Ivan Milisavljevic](https://chat.stackoverflow.com//users/5498855)

[![](assets/1658805810-7b40d4c9b7bb9e4496852db6f508840b.jpg "Ivan Milisavljevic: 7 hours ago")](https://chat.stackoverflow.com//users/5498855)[![](assets/1658805810-19eb15985148a6020c03cbda39156bd8.png "JBis: 9 hours ago")](https://chat.stackoverflow.com//users/7886229)[![](assets/1658805810-83364b8910c7c88236323aa7aaaef3fe.png "Raghav Sood: 13 hours ago")](https://chat.stackoverflow.com//users/1069068)[![](assets/1658805810-334efec10b0d0b0c2c591a62ead0ffc3.png "Mauker: 16 hours ago")](https://chat.stackoverflow.com//users/4070469)[![](assets/1658805810-c666b6e6aeebf5736bdf1d0f45c2cf7f.png "nyconing: Jul 5 at 15:31")](https://chat.stackoverflow.com//users/5858238)[![](assets/1658805810-359f587fc0bd1070470be66b8cb1c075.png "JamesBot: Apr 11 at 20:18")](https://chat.stackoverflow.com//users/11518920)[![](assets/1658805810-50e22e36a5e2d6721a64c8968a24a83d.jpg "chornge")](https://chat.stackoverflow.com//users/1008011)

#### Linked

[

1

](https://stackoverflow.com/q/60562734?lq=1 "Question score (upvotes - downvotes)")[Replace URL fields by Uri with Jackson](https://stackoverflow.com/questions/60562734/replace-url-fields-by-uri-with-jackson?noredirect=1&lq=1)

[

1

](https://stackoverflow.com/q/63475238?lq=1 "Question score (upvotes - downvotes)")[How to solve conflicting getter definitions for property in jackson without access to source](https://stackoverflow.com/questions/63475238/how-to-solve-conflicting-getter-definitions-for-property-in-jackson-without-acce?noredirect=1&lq=1)

[

1

](https://stackoverflow.com/q/61337225?lq=1 "Question score (upvotes - downvotes)")[Is there a custom Json Serializer pattern to serialize a property differently including the property's key name?](https://stackoverflow.com/questions/61337225/is-there-a-custom-json-serializer-pattern-to-serialize-a-property-differently-in?noredirect=1&lq=1)

[

1

](https://stackoverflow.com/q/58076716?lq=1 "Question score (upvotes - downvotes)")[Custom annotation based ignore field from serialisation](https://stackoverflow.com/questions/58076716/custom-annotation-based-ignore-field-from-serialisation?noredirect=1&lq=1)

#### Related

[

338

](https://stackoverflow.com/q/8367312?rq=1 "Question score (upvotes - downvotes)")[Serializing with Jackson (JSON) - getting "No serializer found"?](https://stackoverflow.com/questions/8367312/serializing-with-jackson-json-getting-no-serializer-found?rq=1)

[

45

](https://stackoverflow.com/q/8837018?rq=1 "Question score (upvotes - downvotes)")[Jackson json deserialization, ignore root element from json](https://stackoverflow.com/questions/8837018/jackson-json-deserialization-ignore-root-element-from-json?rq=1)

[

2

](https://stackoverflow.com/q/18580449?rq=1 "Question score (upvotes - downvotes)")[Ignoring properties and accessors while using jackson](https://stackoverflow.com/questions/18580449/ignoring-properties-and-accessors-while-using-jackson?rq=1)

[

6

](https://stackoverflow.com/q/18703831?rq=1 "Question score (upvotes - downvotes)")[jackson annotations being ignored](https://stackoverflow.com/questions/18703831/jackson-annotations-being-ignored?rq=1)

[

0

](https://stackoverflow.com/q/26170872?rq=1 "Question score (upvotes - downvotes)")[how to only serialize properties annotated with a custom annotation](https://stackoverflow.com/questions/26170872/how-to-only-serialize-properties-annotated-with-a-custom-annotation?rq=1)

[

23

](https://stackoverflow.com/q/27876027?rq=1 "Question score (upvotes - downvotes)")[JSON Jackson - exception when serializing a polymorphic class with custom serializer](https://stackoverflow.com/questions/27876027/json-jackson-exception-when-serializing-a-polymorphic-class-with-custom-serial?rq=1)

[

5

](https://stackoverflow.com/q/35591120?rq=1 "Question score (upvotes - downvotes)")[ignoring unknown fields on JSON objects using jackson](https://stackoverflow.com/questions/35591120/ignoring-unknown-fields-on-json-objects-using-jackson?rq=1)

[

17

](https://stackoverflow.com/q/44671154?rq=1 "Question score (upvotes - downvotes)")[Jackson filtering out fields without annotations](https://stackoverflow.com/questions/44671154/jackson-filtering-out-fields-without-annotations?rq=1)

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [What is "power cut"?](https://stats.stackexchange.com/questions/583138/what-is-power-cut)
*   [Would RTGs be more effective on Titan than in space?](https://worldbuilding.stackexchange.com/questions/233235/would-rtgs-be-more-effective-on-titan-than-in-space)
*   [What characteristics define the "arcade" genre or style of console games?](https://gaming.stackexchange.com/questions/398347/what-characteristics-define-the-arcade-genre-or-style-of-console-games)
*   [How do I best simulate a 80386?](https://retrocomputing.stackexchange.com/questions/24894/how-do-i-best-simulate-a-80386)
*   [Does flying a small GA aircraft use less fuel than driving the same distance by car?](https://skeptics.stackexchange.com/questions/53593/does-flying-a-small-ga-aircraft-use-less-fuel-than-driving-the-same-distance-by)
*   [Is a sha256 hash of a unix timestamp a strong password](https://security.stackexchange.com/questions/263582/is-a-sha256-hash-of-a-unix-timestamp-a-strong-password)
*   [Do you need to add a day to travel insurance if your plane arrives a day later than departure?](https://travel.stackexchange.com/questions/175158/do-you-need-to-add-a-day-to-travel-insurance-if-your-plane-arrives-a-day-later-t)
*   [Rearrange to a palindrome](https://codegolf.stackexchange.com/questions/250283/rearrange-to-a-palindrome)
*   [why does my arrow turns into this strange shape in Inkscape?](https://graphicdesign.stackexchange.com/questions/157971/why-does-my-arrow-turns-into-this-strange-shape-in-inkscape)
*   [Is calling emergency service for someone ever mandatory?](https://law.stackexchange.com/questions/82477/is-calling-emergency-service-for-someone-ever-mandatory)
*   [Multiply numbers by their depth](https://codegolf.stackexchange.com/questions/250242/multiply-numbers-by-their-depth)
*   [License allowing mixing code only with code licensed with any of the OSI-approved licenses, getting "infected" by these licenses (kind of reverse GPL)](https://opensource.stackexchange.com/questions/13064/license-allowing-mixing-code-only-with-code-licensed-with-any-of-the-osi-approve)
*   [How to compute this moment of a bivariate normal distribution?](https://stats.stackexchange.com/questions/583124/how-to-compute-this-moment-of-a-bivariate-normal-distribution)
*   [Fill Curve flattens the curve](https://blender.stackexchange.com/questions/270275/fill-curve-flattens-the-curve)
*   [What's another word for agreeing with another person just for the sake of it?](https://english.stackexchange.com/questions/592386/whats-another-word-for-agreeing-with-another-person-just-for-the-sake-of-it)
*   [Exponential transform of an integer sequence](https://codegolf.stackexchange.com/questions/250261/exponential-transform-of-an-integer-sequence)
*   [Why Heisenberg's Uncertainty Principle?](https://physics.stackexchange.com/questions/720027/why-heisenbergs-uncertainty-principle)
*   [What exactly is a “European style of single-source lighting, which involved the use of incense burners?”](https://movies.stackexchange.com/questions/118170/what-exactly-is-a-european-style-of-single-source-lighting-which-involved-the)
*   [I found this message attached to a pipe in a house I bought, what does it mean](https://diy.stackexchange.com/questions/253495/i-found-this-message-attached-to-a-pipe-in-a-house-i-bought-what-does-it-mean)
*   [Why can't I grammatically repeat the object with the pronoun "it"?](https://ell.stackexchange.com/questions/319556/why-cant-i-grammatically-repeat-the-object-with-the-pronoun-it)
*   [A long time ago, on a specific day of the week](https://scifi.stackexchange.com/questions/265933/a-long-time-ago-on-a-specific-day-of-the-week)
*   [How is vector<vector<int>> "heavier" than vector<pair<int,int>>?](https://stackoverflow.com/questions/73095254/how-is-vectorvectorint-heavier-than-vectorpairint-int)
*   [Ancient Rome family names still being used: how did they survive?](https://history.stackexchange.com/questions/69488/ancient-rome-family-names-still-being-used-how-did-they-survive)
*   [Way to measure delay propagation between two different systems](https://electronics.stackexchange.com/questions/628774/way-to-measure-delay-propagation-between-two-different-systems)

[Question feed](https://stackoverflow.com/feeds/question/58763305 "Feed of this question and its answers")