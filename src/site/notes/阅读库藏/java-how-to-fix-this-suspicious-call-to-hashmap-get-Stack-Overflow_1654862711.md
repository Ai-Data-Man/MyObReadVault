---
{"title":"java - how to fix this suspicious call to hashmap.get - Stack Overflow","url":"https://stackoverflow.com/questions/61264067/how-to-fix-this-suspicious-call-to-hashmap-get","clipped_at":"2022-06-10 20:05:11","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/java-how-to-fix-this-suspicious-call-to-hashmap-get-Stack-Overflow_1654862711/","dgPassFrontmatter":true}
---


# java - how to fix this suspicious call to hashmap.get - Stack Overflow

# [how to fix this suspicious call to hashmap.get](https://stackoverflow.com/questions/61264067/how-to-fix-this-suspicious-call-to-hashmap-get)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 2 years, 1 month ago

Modified [2 years, 1 month ago](https://stackoverflow.com/questions/61264067/how-to-fix-this-suspicious-call-to-hashmap-get?lastactivity "2020-04-17 04:26:01Z")

Viewed 6k times

3

[](https://stackoverflow.com/posts/61264067/timeline)

so this is my HashMap

```java
        HashMap<Literal, Double> literalHashMap = new HashMap<>();
        for(Literal literal : literalHashSet) {
            if(literal.isTrue(node.state)){ literalHashMap.put(literal, 0.0);}
            else{literalHashMap.put(literal, Double.MAX_VALUE);}
        }
```

then when I do this

```java
literalHashMap.put((Literal)proposition, updatedCost);
```

but when I do this

```java
cost += literalHashMap.get(proposition)
```

I see a warning saying suspicious call to hashmap.get

any idea how to fix that?

thank you

[java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [hashmap](https://stackoverflow.com/questions/tagged/hashmap "show questions tagged 'hashmap'")

[Share](https://stackoverflow.com/q/61264067 "Short permalink to this question")

Follow

asked Apr 17, 2020 at 4:16

user13283790

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 1 Answer

Sorted by:

Highest score (default) Date modified (newest first) Date created (oldest first)

12

[](https://stackoverflow.com/posts/61264158/timeline)

It's saying suspicious call because you pass a `Literal` Object as the key (`literalHashMap.put((Literal)proposition, updatedCost);` you downcast proposition so it becomes a `Literal` object). So, I can assume that `proposition` is not a `Literal` object. However, you pass `proposition` as the key to the `HashMap`, which is a different kind of object entirely. To fix this: `cost += literalHashMap.get((Literal) proposition);`. Or, to save a _tiny_ bit of time:

```java
Literal lit = (Literal) proposition;
literalHashMap.put((lit, updatedCost);
cost += literalHashMap.get(list);
```

To avoid downcasting twice.

[Share](https://stackoverflow.com/a/61264158 "Short permalink to this answer")

Follow

answered Apr 17, 2020 at 4:26

[

![user avatar](/img/user/阅读库藏/assets/1654862711-ccfca58655d11a81445eb5a42e2848c0.png)

](https://stackoverflow.com/users/13173633/higigig)

[Higigig](https://stackoverflow.com/users/13173633/higigig)

1,07711 gold badge66 silver badges2424 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

  

## Your Answer

### Sign up or [log in](https://stackoverflow.com/users/login?ssrc=question_page&returnurl=https%3a%2f%2fstackoverflow.com%2fquestions%2f61264067%2fhow-to-fix-this-suspicious-call-to-hashmap-get%23new-answer)

Sign up using Google

Sign up using Facebook

Sign up using Email and Password

 

### Post as a guest

Name

Email

Required, but never shown

Post Your Answer

By clicking “Post Your Answer”, you agree to our [terms of service](https://stackoverflow.com/legal/terms-of-service/public), [privacy policy](https://stackoverflow.com/legal/privacy-policy) and [cookie policy](https://stackoverflow.com/legal/cookie-policy)

The Overflow Blog

*   [The Great Decentralization? Geographic shifts and where tech talent is moving...](https://stackoverflow.blog/2022/06/08/the-great-decentralization-geographic-shifts-and-where-tech-talent-is-moving-next/?cb=1 "The Great Decentralization? Geographic shifts and where tech talent is moving next")
    
*   [Want to be great at UX research? Take a cue from cultural anthropology (Ep. 451)](https://stackoverflow.blog/2022/06/10/want-to-be-great-at-ux-research-take-a-cue-from-cultural-anthropology-ep-451/?cb=1)
    

Featured on Meta

*   [Announcing the arrival of Valued Associate #1214: Dalmarus](https://meta.stackexchange.com/questions/378862/announcing-the-arrival-of-valued-associate-1214-dalmarus?cb=1)
    
*   [Improvements to site status and incident communication](https://meta.stackexchange.com/questions/378941/improvements-to-site-status-and-incident-communication?cb=1)
    
*   [Collectives Update: Introducing Bulletins](https://meta.stackoverflow.com/questions/418333/collectives-update-introducing-bulletins?cb=1)
    
*   [The \[comma\] tag is being burninated](https://meta.stackoverflow.com/questions/405831/the-comma-tag-is-being-burninated?cb=1)
    
*   [Ask Wizard Test Results and Next Steps](https://meta.stackoverflow.com/questions/418491/ask-wizard-test-results-and-next-steps?cb=1)
    

#### Related

[

3771

](https://stackoverflow.com/q/46898?rq=1 "Question score (upvotes - downvotes)")[How do I efficiently iterate over each entry in a Java Map?](https://stackoverflow.com/questions/46898/how-do-i-efficiently-iterate-over-each-entry-in-a-java-map?rq=1)

[

2543

](https://stackoverflow.com/q/285177?rq=1 "Question score (upvotes - downvotes)")[How do I call one constructor from another in Java?](https://stackoverflow.com/questions/285177/how-do-i-call-one-constructor-from-another-in-java?rq=1)

[

4521

](https://stackoverflow.com/q/309424?rq=1 "Question score (upvotes - downvotes)")[How do I read / convert an InputStream into a String in Java?](https://stackoverflow.com/questions/309424/how-do-i-read-convert-an-inputstream-into-a-string-in-java?rq=1)

[

3914

](https://stackoverflow.com/q/363681?rq=1 "Question score (upvotes - downvotes)")[How do I generate random integers within a specific range in Java?](https://stackoverflow.com/questions/363681/how-do-i-generate-random-integers-within-a-specific-range-in-java?rq=1)

[

673

](https://stackoverflow.com/q/509076?rq=1 "Question score (upvotes - downvotes)")[How do I address unchecked cast warnings?](https://stackoverflow.com/questions/509076/how-do-i-address-unchecked-cast-warnings?rq=1)

[

856

](https://stackoverflow.com/q/2745265?rq=1 "Question score (upvotes - downvotes)")[Is List<Dog> a subclass of List<Animal>? Why are Java generics not implicitly polymorphic?](https://stackoverflow.com/questions/2745265/is-listdog-a-subclass-of-listanimal-why-are-java-generics-not-implicitly-po?rq=1)

[

3362

](https://stackoverflow.com/q/5585779?rq=1 "Question score (upvotes - downvotes)")[How do I convert a String to an int in Java?](https://stackoverflow.com/questions/5585779/how-do-i-convert-a-string-to-an-int-in-java?rq=1)

[

2630

](https://stackoverflow.com/q/6343166?rq=1 "Question score (upvotes - downvotes)")[How can I fix 'android.os.NetworkOnMainThreadException'?](https://stackoverflow.com/questions/6343166/how-can-i-fix-android-os-networkonmainthreadexception?rq=1)

[

3588

](https://stackoverflow.com/q/6470651?rq=1 "Question score (upvotes - downvotes)")[How can I create a memory leak in Java?](https://stackoverflow.com/questions/6470651/how-can-i-create-a-memory-leak-in-java?rq=1)

[

1676

](https://stackoverflow.com/q/10382929?rq=1 "Question score (upvotes - downvotes)")[How to fix java.lang.UnsupportedClassVersionError: Unsupported major.minor version](https://stackoverflow.com/questions/10382929/how-to-fix-java-lang-unsupportedclassversionerror-unsupported-major-minor-versi?rq=1)

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [Two players toss a coin; probability game doesn't end in 100 tosses?](https://math.stackexchange.com/questions/4469203/two-players-toss-a-coin-probability-game-doesnt-end-in-100-tosses)
*   [The Game of Golden Squares](https://puzzling.stackexchange.com/questions/116550/the-game-of-golden-squares)
*   [Is requiring a link to my site allowed for a FOSS license?](https://opensource.stackexchange.com/questions/12907/is-requiring-a-link-to-my-site-allowed-for-a-foss-license)
*   [Has the small meteorite that hit Webb done a lot of damage?](https://astronomy.stackexchange.com/questions/49532/has-the-small-meteorite-that-hit-webb-done-a-lot-of-damage)
*   [Are there any bacteria that could be used as food?](https://biology.stackexchange.com/questions/108415/are-there-any-bacteria-that-could-be-used-as-food)

[more hot questions](#)

[Question feed](https://stackoverflow.com/feeds/question/61264067 "Feed of this question and its answers")