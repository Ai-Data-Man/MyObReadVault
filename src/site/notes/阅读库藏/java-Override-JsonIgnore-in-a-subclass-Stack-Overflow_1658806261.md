---
{"title":"java - Override @JsonIgnore in a subclass - Stack Overflow","url":"https://stackoverflow.com/questions/37074711/override-jsonignore-in-a-subclass","clipped_at":"2022-07-26 11:31:01","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/java-Override-JsonIgnore-in-a-subclass-Stack-Overflow_1658806261/","dgPassFrontmatter":true}
---


# java - Override @JsonIgnore in a subclass - Stack Overflow

# [Override @JsonIgnore in a subclass](https://stackoverflow.com/questions/37074711/override-jsonignore-in-a-subclass)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 6 years, 2 months ago

Modified [6 years, 2 months ago](https://stackoverflow.com/questions/37074711/override-jsonignore-in-a-subclass?lastactivity "2016-05-06 15:32:27Z")

Viewed 9k times

7

1

[](https://stackoverflow.com/posts/37074711/timeline)

I have the following abstract class:

```java
public abstract class StandardTimeStamp {

  @Temporal(TemporalType.TIMESTAMP)
  @Column(nullable = false)
  @JsonIgnore
  private Date lastUpdated;

  @PreUpdate
  public void generatelastUpdated() {
    this.lastUpdated = new Date();
  }

  public Date getLastUpdated() {
    return lastUpdated;
  }

  public void setLastUpdated(Date lastUpdated) {
    this.lastUpdated = lastUpdated;
  }
}
```

I have an entity that is a subclass in which I need the lastUpdate value to be sent to the browser, so I need to override the @JsonIgnore. I have tried a few things, the following code being one of them:

```java
public class MyEntity extends StandardTimestamp implements Serializable {

  @Column(name = "LastUpdated")
  private Date lastUpdated;

  @JsonIgnore(false)
  @Override
  public Date getLastUpdated() {
    return lastUpdated;
  }
}
```

This does add the lastUpdated attribute on the JSON response, however its value is null when it is not null in the database. I have other Dates in the subclass that work, but they aren't hidden with @IgnoreJson in the super class.

Any thoughts?

[java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [inheritance](https://stackoverflow.com/questions/tagged/inheritance "show questions tagged 'inheritance'") [jackson](https://stackoverflow.com/questions/tagged/jackson "show questions tagged 'jackson'")

[Share](https://stackoverflow.com/q/37074711 "Short permalink to this question")

[Improve this question](https://stackoverflow.com/posts/37074711/edit)

Follow

asked May 6, 2016 at 14:15

[

![user avatar](assets/1658806261-6ac548e9b4a6d5433c2178c4f50f8450.png)

](https://stackoverflow.com/users/2563429/nick-suwyn)

[Nick Suwyn](https://stackoverflow.com/users/2563429/nick-suwyn)

46111 gold badge55 silver badges2121 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 1 Answer

Sorted by:

Highest score (default) Trending (recent votes count more) Date modified (newest first) Date created (oldest first)

15

[](https://stackoverflow.com/posts/37076241/timeline)

You're introducing a second field named `lastUpdated` with this declaration

```java
@Column(name = "LastUpdated")
private Date lastUpdated;
```

in your subtype `MyEntity`. I'm not sure how the database mapper handles this, but it seems redundant to me.

I can see two options.

One, get rid of this new field and have your overriden `getLastUpdated` method delegate to its super implementation. For example

```java
@Override
@JsonIgnore(false)
@JsonProperty
public Date getLastUpdated() {
    return super.getLastUpdated();
}
```

When serializing a `MyEntity` object, Jackson will discover a single property through this accessor, `lastUpdated`, and use it to generate the JSON. It'll end up retrieving it from the superclass.

Two, use the class annotation `@JsonIgnoreProperties` to mark the ignored properties in the superclass

```java
@JsonIgnoreProperties("lastUpdated")
abstract class StandardTimeStamp {
```

then clear (or add different values to) that "list" in the subclass

```java
@JsonIgnoreProperties()
class MyEntity extends StandardTimeStamp implements Serializable {
```

Jackson will only use the annotation on the subclass, if it's present, and ignore the annotation on the superclass.

[Share](https://stackoverflow.com/a/37076241 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/37076241/edit)

Follow

answered May 6, 2016 at 15:32

[

![user avatar](assets/1658806261-e1800c59c4b5fd67847c5f3ab08304be.png)

](https://stackoverflow.com/users/438154/sotirios-delimanolis)

[Sotirios Delimanolis](https://stackoverflow.com/users/438154/sotirios-delimanolis)

265k5656 gold badges675675 silver badges705705 bronze badges

*   This works, however I am still getting null in the JSON attribute on the front-end, but I think that is on the database side. I will keep looking into that. Thanks for the great answer! :)
    
    – [Nick Suwyn](https://stackoverflow.com/users/2563429/nick-suwyn "461 reputation")
    
    [May 6, 2016 at 15:42](#comment61697407_37076241)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

  

## Your Answer

### Sign up or [log in](https://stackoverflow.com/users/login?ssrc=question_page&returnurl=https%3a%2f%2fstackoverflow.com%2fquestions%2f37074711%2foverride-jsonignore-in-a-subclass%23new-answer)

Sign up using Google

Sign up using Facebook

Sign up using Email and Password

 

### Post as a guest

Name

Email

Required, but never shown

Post Your Answer

By clicking “Post Your Answer”, you agree to our [terms of service](https://stackoverflow.com/legal/terms-of-service/public), [privacy policy](https://stackoverflow.com/legal/privacy-policy) and [cookie policy](https://stackoverflow.com/legal/cookie-policy)

## Not the answer you're looking for? Browse other questions tagged [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [inheritance](https://stackoverflow.com/questions/tagged/inheritance "show questions tagged 'inheritance'") [jackson](https://stackoverflow.com/questions/tagged/jackson "show questions tagged 'jackson'") or [ask your own question](https://stackoverflow.com/questions/ask).

The Overflow Blog

*   [Game Boy emulators, PowerPoint developers, and the enduring appeal of Pokémon...](https://stackoverflow.blog/2022/07/22/game-boy-emulators-powerpoint-developers-and-the-enduring-appeal-of-pokemon-go-ep-466/?cb=1 "Game Boy emulators, PowerPoint developers, and the enduring appeal of Pokémon GO (Ep. 466)")
    
*   [Automate the boring parts of your job](https://stackoverflow.blog/2022/07/25/automate-the-boring-parts-of-your-job/?cb=1)
    

Featured on Meta

*   [Announcing the Stacks Editor Beta release!](https://meta.stackexchange.com/questions/380295/announcing-the-stacks-editor-beta-release?cb=1)
    
*   [Trending: A new answer sorting option](https://meta.stackoverflow.com/questions/418767/trending-a-new-answer-sorting-option?cb=1)
    
*   [The \[options\] tag is being burninated](https://meta.stackoverflow.com/questions/417320/the-options-tag-is-being-burninated?cb=1)
    

#### Related

[

1294

](https://stackoverflow.com/q/1678122?rq=1 "Question score (upvotes - downvotes)")['Must Override a Superclass Method' Errors after importing a project into Eclipse](https://stackoverflow.com/questions/1678122/must-override-a-superclass-method-errors-after-importing-a-project-into-eclips?rq=1)

[

860

](https://stackoverflow.com/q/2745265?rq=1 "Question score (upvotes - downvotes)")[Is List<Dog> a subclass of List<Animal>? Why are Java generics not implicitly polymorphic?](https://stackoverflow.com/questions/2745265/is-listdog-a-subclass-of-listanimal-why-are-java-generics-not-implicitly-po?rq=1)

[

412

](https://stackoverflow.com/q/12505141?rq=1 "Question score (upvotes - downvotes)")[Only using @JsonIgnore during serialization, but not deserialization](https://stackoverflow.com/questions/12505141/only-using-jsonignore-during-serialization-but-not-deserialization?rq=1)

[

3

](https://stackoverflow.com/q/13548515?rq=1 "Question score (upvotes - downvotes)")[JPA : OpenJPA : The id class specified by type does not match the primary key fields of the class](https://stackoverflow.com/questions/13548515/jpa-openjpa-the-id-class-specified-by-type-does-not-match-the-primary-key-fi?rq=1)

[

1

](https://stackoverflow.com/q/13916629?rq=1 "Question score (upvotes - downvotes)")[Cannot override abstract method of superclass when subclass is in another package](https://stackoverflow.com/questions/13916629/cannot-override-abstract-method-of-superclass-when-subclass-is-in-another-packag?rq=1)

[

531

](https://stackoverflow.com/q/18777989?rq=1 "Question score (upvotes - downvotes)")[How should I have explained the difference between an Interface and an Abstract class?](https://stackoverflow.com/questions/18777989/how-should-i-have-explained-the-difference-between-an-interface-and-an-abstract?rq=1)

[

3

](https://stackoverflow.com/q/26067831?rq=1 "Question score (upvotes - downvotes)")[Method override returns null](https://stackoverflow.com/questions/26067831/method-override-returns-null?rq=1)

[

10

](https://stackoverflow.com/q/36660635?rq=1 "Question score (upvotes - downvotes)")[swift: How can I override a public method in superclass to be a private method in subclass](https://stackoverflow.com/questions/36660635/swift-how-can-i-override-a-public-method-in-superclass-to-be-a-private-method-i?rq=1)

[

1

](https://stackoverflow.com/q/65604399?rq=1 "Question score (upvotes - downvotes)")[jackson JsonInclude Non\_Null not ignoring working on nested Object Properties](https://stackoverflow.com/questions/65604399/jackson-jsoninclude-non-null-not-ignoring-working-on-nested-object-properties?rq=1)

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [Can I decline a PhD offer after deferral?](https://academia.stackexchange.com/questions/187257/can-i-decline-a-phd-offer-after-deferral)
*   [A long time ago, on a specific day of the week](https://scifi.stackexchange.com/questions/265933/a-long-time-ago-on-a-specific-day-of-the-week)
*   [Pro's and Con's of a starship with a tractor configuration?](https://worldbuilding.stackexchange.com/questions/233194/pros-and-cons-of-a-starship-with-a-tractor-configuration)
*   [Ton vs Klang vs Geräusch for sounds electronic and electrical devices make?](https://german.stackexchange.com/questions/71255/ton-vs-klang-vs-ger%c3%a4usch-for-sounds-electronic-and-electrical-devices-make)
*   [Was the Force stronger in the past?](https://scifi.stackexchange.com/questions/265892/was-the-force-stronger-in-the-past)
*   ["sort -h" not working correctly](https://unix.stackexchange.com/questions/711159/sort-h-not-working-correctly)
*   [Does June 2022 LaTeX require use of padmanagement-testphase and hyperref key pdfproducer?](https://tex.stackexchange.com/questions/651965/does-june-2022-latex-require-use-of-padmanagement-testphase-and-hyperref-key-pdf)
*   [Is there anything currently 46 billion light years away from Earth that we can see?](https://astronomy.stackexchange.com/questions/50011/is-there-anything-currently-46-billion-light-years-away-from-earth-that-we-can-s)
*   [Why do we rely on computers in critical fields?](https://cs.stackexchange.com/questions/153158/why-do-we-rely-on-computers-in-critical-fields)
*   [Why do people experience weightlessness on the way up in parabolic flight?](https://space.stackexchange.com/questions/59825/why-do-people-experience-weightlessness-on-the-way-up-in-parabolic-flight)
*   [How to repair the handle of a mitre saw](https://diy.stackexchange.com/questions/253505/how-to-repair-the-handle-of-a-mitre-saw)
*   [Is it legal to have one breaker control four rooms?](https://diy.stackexchange.com/questions/253520/is-it-legal-to-have-one-breaker-control-four-rooms)
*   [Am I allowed to videotape (dashcam) when entering the United States by car?](https://travel.stackexchange.com/questions/175141/am-i-allowed-to-videotape-dashcam-when-entering-the-united-states-by-car)
*   [How to handle deception skill rolls fairly based on the premise of the deception?](https://rpg.stackexchange.com/questions/200144/how-to-handle-deception-skill-rolls-fairly-based-on-the-premise-of-the-deception)
*   [I'm trying to do a Charisma (Persuasion) check with no DC to compare it to!](https://rpg.stackexchange.com/questions/200190/im-trying-to-do-a-charisma-persuasion-check-with-no-dc-to-compare-it-to)
*   [Why Heisenberg's Uncertainty Principle?](https://physics.stackexchange.com/questions/720027/why-heisenbergs-uncertainty-principle)
*   [Can I get the tourist visa printed on any passport I want?](https://travel.stackexchange.com/questions/175164/can-i-get-the-tourist-visa-printed-on-any-passport-i-want)
*   [On the translation of the first hypothesis of Theaetetus (Theaetetus, 151e)](https://latin.stackexchange.com/questions/18573/on-the-translation-of-the-first-hypothesis-of-theaetetus-theaetetus-151e)
*   [How does dmesg differentiate between different log levels?](https://serverfault.com/questions/1106517/how-does-dmesg-differentiate-between-different-log-levels)
*   [Meaning of the word "sons" in "Our God, Our Help in Ages Past"?](https://christianity.stackexchange.com/questions/92003/meaning-of-the-word-sons-in-our-god-our-help-in-ages-past)
*   [Ideas for a human convenient, long term date system spanning many trillions of years](https://worldbuilding.stackexchange.com/questions/233136/ideas-for-a-human-convenient-long-term-date-system-spanning-many-trillions-of-y)
*   [Why is the Moon's gravity so high compared to its mass?](https://physics.stackexchange.com/questions/720033/why-is-the-moons-gravity-so-high-compared-to-its-mass)
*   [What's another word for agreeing with another person just for the sake of it?](https://english.stackexchange.com/questions/592386/whats-another-word-for-agreeing-with-another-person-just-for-the-sake-of-it)
*   [Why ether transfer sometimes use more that 21000 gas?](https://ethereum.stackexchange.com/questions/132341/why-ether-transfer-sometimes-use-more-that-21000-gas)

[Question feed](https://stackoverflow.com/feeds/question/37074711 "Feed of this question and its answers")