---
{"title":"bigdecimal - What does big decimal precision zero mean in java - Stack Overflow","url":"https://stackoverflow.com/questions/39761265/what-does-big-decimal-precision-zero-mean-in-java","clipped_at":"2022-07-29 08:58:52","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/bigdecimal-What-does-big-decimal-precision-zero-mean-in-java-Stack-Overflow_1659056332/","dgPassFrontmatter":true}
---


# bigdecimal - What does big decimal precision zero mean in java - Stack Overflow

# [What does big decimal precision zero mean in java](https://stackoverflow.com/questions/39761265/what-does-big-decimal-precision-zero-mean-in-java)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 5 years, 10 months ago

Modified [5 years, 10 months ago](https://stackoverflow.com/questions/39761265/what-does-big-decimal-precision-zero-mean-in-java?lastactivity "2016-09-29 12:34:15Z")

Viewed 1k times

0

[](https://stackoverflow.com/posts/39761265/timeline)

I treated 500 as BigDecimal in java. And i turned into `{･UnscaledValue = 5 ･Scale = -2 ･Precision = 0}` I really don't understand why precision is 0. I thought it might be 1. Can anyone explain?

[java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [bigdecimal](https://stackoverflow.com/questions/tagged/bigdecimal "show questions tagged 'bigdecimal'")

[Share](https://stackoverflow.com/q/39761265 "Short permalink to this question")

Follow

[edited Sep 29, 2016 at 12:34](https://stackoverflow.com/posts/39761265/revisions "show all edits to this post")

[

![user avatar](/img/user/阅读库藏/assets/1659056332-3c2f47a58b2ff0cd1f9fe13d20082dd9.png)

](https://stackoverflow.com/users/5772882/ole-v-v)

[Ole V.V.](https://stackoverflow.com/users/5772882/ole-v-v)

76.9k1515 gold badges121121 silver badges146146 bronze badges

asked Sep 29, 2016 at 4:09

[

![user avatar](/img/user/阅读库藏/assets/1659056332-829a351f1bcd91afe893259c930980a0.png)

](https://stackoverflow.com/users/6897253/aki)

[Aki](https://stackoverflow.com/users/6897253/aki)

922 bronze badges

*   4
    
    Show us your code plz .
    
    – [passion](https://stackoverflow.com/users/6768947/passion "1,220 reputation")
    
    [Sep 29, 2016 at 4:23](#comment66818130_39761265)
    
*   Is there a reason why you need to know, or just curious (I’d be too)?
    
    – [Ole V.V.](https://stackoverflow.com/users/5772882/ole-v-v "76,933 reputation")
    
    [Sep 29, 2016 at 5:02](#comment66818818_39761265)
    
*   1
    
    I'm voting to close this question because, without code, it's unclear what you're asking. This could be a good question, but it needs code.
    
    – [Chris Martin](https://stackoverflow.com/users/402884/chris-martin "29,574 reputation")
    
    [Sep 29, 2016 at 15:34](#comment66843238_39761265)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 1 Answer

Sorted by:

Highest score (default) Trending (recent votes count more) Date modified (newest first) Date created (oldest first)

2

[](https://stackoverflow.com/posts/39761855/timeline)

The precision for 500, constructed in the most straightforward way, is 3.

```java
scala> new java.math.BigDecimal("500").precision()
res0: Int = 3
```

If you did something different, you'll have to show the code so we know what you did.

The meaning of `precision` from [the documentation](https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html#precision--):

> The precision is the number of digits in the unscaled value. The precision of a zero value is 1.

[Share](https://stackoverflow.com/a/39761855 "Short permalink to this answer")

Follow

answered Sep 29, 2016 at 5:10

[

![user avatar](/img/user/阅读库藏/assets/1659056332-0adbc2edc78adbb7673617a9536fe206.jpg)

](https://stackoverflow.com/users/402884/chris-martin)

[Chris Martin](https://stackoverflow.com/users/402884/chris-martin)

29.6k88 gold badges7474 silver badges131131 bronze badges

*   If the scale is -2, and the unscaledValue is 5, then it is obviously stored as 5 \* 10^2. Then, the precision should be 1, not 0, nor 3.
    
    – [Rudy Velthuis](https://stackoverflow.com/users/95954/rudy-velthuis "27,984 reputation")
    
    [Sep 29, 2016 at 7:05](#comment66822142_39761855)
    
*   I too get 3. I also tried `new BigDecimal(500.0)`, the presicion is still 3 and the scale 0. I still haven’t figured out how the OP could possibly get a precision of 0.
    
    – [Ole V.V.](https://stackoverflow.com/users/5772882/ole-v-v "76,933 reputation")
    
    [Sep 29, 2016 at 8:37](#comment66825466_39761855)
    
*   @OleV.V.: Don't instantiate a BigDecimal from a floating point type (double). But if you do, say, `x = new BigDecimal("5e2")` or something like `x = new BigDecimal("5").movePointRight(2);`, or from a simple calucaltion, you can get an `unscaledValue` of `5` and a `scale` of `-2`. So the only problem is: why is the precision `0`. It should be `1`, by definition.
    
    – [Rudy Velthuis](https://stackoverflow.com/users/95954/rudy-velthuis "27,984 reputation")
    
    [Sep 29, 2016 at 13:30](#comment66837784_39761855)
    
*   What I am trying to say is: _it does not matter how he got an `unscaledValue`of `5` and a `scale` of `-2`. It is a valid BigDecimal denoting the number 500, and its `precision()` should **not** be `3`, it should be `1`._
    
    – [Rudy Velthuis](https://stackoverflow.com/users/95954/rudy-velthuis "27,984 reputation")
    
    [Sep 29, 2016 at 13:33](#comment66837938_39761855)
    
*   Thanks for the suggestions, @RudyVelthuis. You clearly know a lot more than I about `BigDecimal`. However, for the precision of 3 I admit I still tend to believe my Java installation more then you (for the precision of 0 I’m starting to believe the OP made a mistake I never actually got a precision of 0 from his `BigDecimal` — now I’m being frank bordering to rudeness).
    
    – [Ole V.V.](https://stackoverflow.com/users/5772882/ole-v-v "76,933 reputation")
    
    [Sep 29, 2016 at 13:41](#comment66838280_39761855)
    

[Show **3** more comments](# "Expand to show all comments on this post")

  

## Your Answer

### Sign up or [log in](https://stackoverflow.com/users/login?ssrc=question_page&returnurl=https%3a%2f%2fstackoverflow.com%2fquestions%2f39761265%2fwhat-does-big-decimal-precision-zero-mean-in-java%23new-answer)

Sign up using Google

Sign up using Facebook

Sign up using Email and Password

 

### Post as a guest

Name

Email

Required, but never shown

Post Your Answer

By clicking “Post Your Answer”, you agree to our [terms of service](https://stackoverflow.com/legal/terms-of-service/public), [privacy policy](https://stackoverflow.com/legal/privacy-policy) and [cookie policy](https://stackoverflow.com/legal/cookie-policy)

## Not the answer you're looking for? Browse other questions tagged [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [bigdecimal](https://stackoverflow.com/questions/tagged/bigdecimal "show questions tagged 'bigdecimal'") or [ask your own question](https://stackoverflow.com/questions/ask).

The Overflow Blog

*   [Always learning](https://stackoverflow.blog/2022/07/27/always-learning/?cb=1)
    
*   [Measurable and meaningful skill levels for developers](https://stackoverflow.blog/2022/07/28/measurable-and-meaningful-skill-levels-for-developers/?cb=1)
    

Featured on Meta

*   [Announcing the Stacks Editor Beta release!](https://meta.stackexchange.com/questions/380295/announcing-the-stacks-editor-beta-release?cb=1)
    
*   [Trending: A new answer sorting option](https://meta.stackoverflow.com/questions/418767/trending-a-new-answer-sorting-option?cb=1)
    
*   [Should we burninate the \[shopping\] and \[shop\] tags?](https://meta.stackoverflow.com/questions/404174/should-we-burninate-the-shopping-and-shop-tags?cb=1)
    

#### Related

[

4177

](https://stackoverflow.com/q/40471?rq=1 "Question score (upvotes - downvotes)")[What are the differences between a HashMap and a Hashtable in Java?](https://stackoverflow.com/questions/40471/what-are-the-differences-between-a-hashmap-and-a-hashtable-in-java?rq=1)

[

2614

](https://stackoverflow.com/q/65035?rq=1 "Question score (upvotes - downvotes)")[Does a finally block always get executed in Java?](https://stackoverflow.com/questions/65035/does-a-finally-block-always-get-executed-in-java?rq=1)

[

3519

](https://stackoverflow.com/q/215497?rq=1 "Question score (upvotes - downvotes)")[What is the difference between public, protected, package-private and private in Java?](https://stackoverflow.com/questions/215497/what-is-the-difference-between-public-protected-package-private-and-private-in?rq=1)

[

1042

](https://stackoverflow.com/q/338206?rq=1 "Question score (upvotes - downvotes)")[Why can't I use switch statement on a String?](https://stackoverflow.com/questions/338206/why-cant-i-use-switch-statement-on-a-string?rq=1)

[

1981

](https://stackoverflow.com/q/997482?rq=1 "Question score (upvotes - downvotes)")[Does Java support default parameter values?](https://stackoverflow.com/questions/997482/does-java-support-default-parameter-values?rq=1)

[

1099

](https://stackoverflow.com/q/1085709?rq=1 "Question score (upvotes - downvotes)")[What does 'synchronized' mean?](https://stackoverflow.com/questions/1085709/what-does-synchronized-mean?rq=1)

[

26520

](https://stackoverflow.com/q/11227809?rq=1 "Question score (upvotes - downvotes)")[Why is processing a sorted array faster than processing an unsorted array?](https://stackoverflow.com/questions/11227809/why-is-processing-a-sorted-array-faster-than-processing-an-unsorted-array?rq=1)

[

1743

](https://stackoverflow.com/q/18093928?rq=1 "Question score (upvotes - downvotes)")[What does "Could not find or load main class" mean?](https://stackoverflow.com/questions/18093928/what-does-could-not-find-or-load-main-class-mean?rq=1)

[

1237

](https://stackoverflow.com/q/24342886?rq=1 "Question score (upvotes - downvotes)")[How to install Java 8 on Mac](https://stackoverflow.com/questions/24342886/how-to-install-java-8-on-mac?rq=1)

[

1402

](https://stackoverflow.com/q/30727515?rq=1 "Question score (upvotes - downvotes)")[Why is executing Java code in comments with certain Unicode characters allowed?](https://stackoverflow.com/questions/30727515/why-is-executing-java-code-in-comments-with-certain-unicode-characters-allowed?rq=1)

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [Greek directly in math mode XeLaTeX instead of PdfLatex](https://tex.stackexchange.com/questions/652272/greek-directly-in-math-mode-xelatex-instead-of-pdflatex)
*   [Plot a phase diagram](https://mathematica.stackexchange.com/questions/271378/plot-a-phase-diagram)
*   [ethics of keeping a gift card you won at a raffle at a conference your company sent you to?](https://workplace.stackexchange.com/questions/186506/ethics-of-keeping-a-gift-card-you-won-at-a-raffle-at-a-conference-your-company-s)
*   [What's the most important life lesson?](https://puzzling.stackexchange.com/questions/117312/whats-the-most-important-life-lesson)
*   [After I chose a preferred referee for a submitted paper, is it un ethical to drop an email to the referee saying that I suggested their name?](https://academia.stackexchange.com/questions/187347/after-i-chose-a-preferred-referee-for-a-submitted-paper-is-it-un-ethical-to-dro)
*   [How to snap points to line](https://gis.stackexchange.com/questions/436976/how-to-snap-points-to-line)
*   [ARUs with on-board compression](https://bioacoustics.stackexchange.com/questions/907/arus-with-on-board-compression)
*   [can we say "I pushed him by his back" the same way we say "I held the cup by its handle"?](https://ell.stackexchange.com/questions/319809/can-we-say-i-pushed-him-by-his-back-the-same-way-we-say-i-held-the-cup-by-its)
*   [How is making a down payment different from getting a smaller loan?](https://money.stackexchange.com/questions/151904/how-is-making-a-down-payment-different-from-getting-a-smaller-loan)
*   [What is the meaning about the IRS saying that "copy A" is scannable but the online version printed is not scannable?](https://money.stackexchange.com/questions/151888/what-is-the-meaning-about-the-irs-saying-that-copy-a-is-scannable-but-the-onli)
*   [Possibility to machine hub cones](https://bicycles.stackexchange.com/questions/85099/possibility-to-machine-hub-cones)
*   [Shifted auto-sum](https://codegolf.stackexchange.com/questions/250338/shifted-auto-sum)
*   [What verb form do we use in "espero que yo <verb>..."?](https://spanish.stackexchange.com/questions/40907/what-verb-form-do-we-use-in-espero-que-yo-verb)

[more hot questions](#)

[Question feed](https://stackoverflow.com/feeds/question/39761265 "Feed of this question and its answers")