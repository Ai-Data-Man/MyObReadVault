---
{"title":"Java validation @min @max annotion for null values","url":"https://stackoverflow.com/questions/46493670/java-validation-min-max-annotion-for-null-values","clipped_at":"2022-06-22 17:08:44","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/Java-validation-min-max-annotion-for-null-values_1655888924/","dgPassFrontmatter":true}
---

# [Java validation @min @max annotion for null values](https://stackoverflow.com/questions/46493670/java-validation-min-max-annotion-for-null-values)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 4 years, 8 months ago

Modified [4 years, 8 months ago](https://stackoverflow.com/questions/46493670/java-validation-min-max-annotion-for-null-values?lastactivity "2017-09-29 17:07:14Z")

Viewed 6k times

6

[](https://stackoverflow.com/posts/46493670/timeline)

I have put validation for the following field,

```java
@Min(1)
@Max(500)
private int length;
```

however, the length isn't a required field but when I didn't give the "length" in the input, I got this error:

```java
"Validation error, message = must be greater than or equal to 1, path = length"
```

Looking at the @min and @max documentation, it says "null element is considered valid". I know that. If @min @max is only for primitive type, then why the documentation mentions "null" element is considered valid? Can someone let me know how to fix the validation problem? Many thanks.

[java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [validation](https://stackoverflow.com/questions/tagged/validation "show questions tagged 'validation'") [max](https://stackoverflow.com/questions/tagged/max "show questions tagged 'max'") [min](https://stackoverflow.com/questions/tagged/min "show questions tagged 'min'")

[Share](https://stackoverflow.com/q/46493670 "Short permalink to this question")

[Improve this question](https://stackoverflow.com/posts/46493670/edit)

Follow

[edited Sep 29, 2017 at 17:07](https://stackoverflow.com/posts/46493670/revisions "show all edits to this post")

asked Sep 29, 2017 at 17:00

[

![user avatar](/img/user/阅读库藏/assets/1655888924-5468deebc84dde8912f4a3c40b53a3d8.png)

](https://stackoverflow.com/users/4521196/jlp)

[jlp](https://stackoverflow.com/users/4521196/jlp)

1,34644 gold badges2828 silver badges4141 bronze badges

*   1
    
    And _when_ do you expect `length` to be `null`?
    
    – [Tom](https://stackoverflow.com/users/3824919/tom "15,564 reputation")
    
    [Sep 29, 2017 at 17:02](#comment79941704_46493670)
    
*   I didn't expect it to be null. I just don't know how to pass the validation when length is not provided. I thought ""null element is considered valid" meant the scenario of "not provided"
    
    – [jlp](https://stackoverflow.com/users/4521196/jlp "1,346 reputation")
    
    [Sep 29, 2017 at 17:03](#comment79941735_46493670)
    
*   2
    
    An `int` can't ever be `null`, it's a primitive type. If you want it to be nullable, you have to make it an `Integer`.
    
    – [Joachim Sauer](https://stackoverflow.com/users/40342/joachim-sauer "292,153 reputation")
    
    [Sep 29, 2017 at 17:04](#comment79941763_46493670)
    
*   When you don't expect it to be null, then why do you quote that line? And the "min" annotation makes it quite clear that you **_can't_** pass validation when you don't provide a proper value.
    
    – [Tom](https://stackoverflow.com/users/3824919/tom "15,564 reputation")
    
    [Sep 29, 2017 at 17:05](#comment79941810_46493670)
    
*   1
    
    _"If min max is only for primitive type"_ - That assumption is wrong.
    
    – [Tom](https://stackoverflow.com/users/3824919/tom "15,564 reputation")
    
    [Sep 29, 2017 at 17:09](#comment79941943_46493670)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 1 Answer

Sorted by:

Highest score (default) Date modified (newest first) Date created (oldest first)

9

[](https://stackoverflow.com/posts/46493750/timeline)

For optional integer values, you may use `Integer` instead of `int`, since an `int` variable cannot be null and will have 0 as a default value. With an `Integer`, length will be null by default and you should be able to pass the validation.

[Share](https://stackoverflow.com/a/46493750 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/46493750/edit)

Follow

answered Sep 29, 2017 at 17:06

[

![user avatar](/img/user/阅读库藏/assets/1655888924-690798d7914665c257cbae42319cc4c4.jpg)

](https://stackoverflow.com/users/8691695/thibault-seisel)

[Thibault Seisel](https://stackoverflow.com/users/8691695/thibault-seisel)

1,05788 silver badges2020 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

  

## Your Answer

### Sign up or [log in](https://stackoverflow.com/users/login?ssrc=question_page&returnurl=https%3a%2f%2fstackoverflow.com%2fquestions%2f46493670%2fjava-validation-min-max-annotion-for-null-values%23new-answer)

Sign up using Google

Sign up using Facebook

Sign up using Email and Password

 

### Post as a guest

Name

Email

Required, but never shown

Post Your Answer

By clicking “Post Your Answer”, you agree to our [terms of service](https://stackoverflow.com/legal/terms-of-service/public), [privacy policy](https://stackoverflow.com/legal/privacy-policy) and [cookie policy](https://stackoverflow.com/legal/cookie-policy)

## Not the answer you're looking for? Browse other questions tagged [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [validation](https://stackoverflow.com/questions/tagged/validation "show questions tagged 'validation'") [max](https://stackoverflow.com/questions/tagged/max "show questions tagged 'max'") [min](https://stackoverflow.com/questions/tagged/min "show questions tagged 'min'") or [ask your own question](https://stackoverflow.com/questions/ask).

The Overflow Blog

*   [What Apple’s WWDC 2022 means for developers](https://stackoverflow.blog/2022/06/20/what-apples-wwdc-2022-means-for-developers/?cb=1)
    
*   [Experts from Stripe and Waymo explain how to craft great documentation (Ep. 455)](https://stackoverflow.blog/2022/06/21/an-engineers-field-guide-to-great-technical-writing-ep-455/?cb=1)
    

Featured on Meta

*   [Announcing the arrival of Valued Associate #1214: Dalmarus](https://meta.stackexchange.com/questions/378862/announcing-the-arrival-of-valued-associate-1214-dalmarus?cb=1)
    
*   [Testing new traffic management tool](https://meta.stackexchange.com/questions/379561/testing-new-traffic-management-tool?cb=1)
    
*   [Ask Wizard Test Results and Next Steps](https://meta.stackoverflow.com/questions/418491/ask-wizard-test-results-and-next-steps?cb=1)
    

#### Related

[

1653

](https://stackoverflow.com/q/85190?rq=1 "Question score (upvotes - downvotes)")[How does the Java 'for each' loop work?](https://stackoverflow.com/questions/85190/how-does-the-java-for-each-loop-work?rq=1)

[

583

](https://stackoverflow.com/q/124417?rq=1 "Question score (upvotes - downvotes)")[Is there a Max function in SQL Server that takes two values like Math.Max in .NET?](https://stackoverflow.com/questions/124417/is-there-a-max-function-in-sql-server-that-takes-two-values-like-math-max-in-ne?rq=1)

[

1974

](https://stackoverflow.com/q/997482?rq=1 "Question score (upvotes - downvotes)")[Does Java support default parameter values?](https://stackoverflow.com/questions/997482/does-java-support-default-parameter-values?rq=1)

[

333

](https://stackoverflow.com/q/1565688?rq=1 "Question score (upvotes - downvotes)")[How to get the max of two values in MySQL?](https://stackoverflow.com/questions/1565688/how-to-get-the-max-of-two-values-in-mysql?rq=1)

[

634

](https://stackoverflow.com/q/2474015?rq=1 "Question score (upvotes - downvotes)")[Getting the index of the returned max or min item using max()/min() on a list](https://stackoverflow.com/questions/2474015/getting-the-index-of-the-returned-max-or-min-item-using-max-min-on-a-list?rq=1)

[

377

](https://stackoverflow.com/q/3437404?rq=1 "Question score (upvotes - downvotes)")[MIN and MAX in C](https://stackoverflow.com/questions/3437404/min-and-max-in-c?rq=1)

[

3

](https://stackoverflow.com/q/33735477?rq=1 "Question score (upvotes - downvotes)")[@Min validation on String](https://stackoverflow.com/questions/33735477/min-validation-on-string?rq=1)

[

1

](https://stackoverflow.com/q/49855980?rq=1 "Question score (upvotes - downvotes)")[How to set custom message for custom validation in CI](https://stackoverflow.com/questions/49855980/how-to-set-custom-message-for-custom-validation-in-ci?rq=1)

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [How is \`std::cout\` implemented?](https://stackoverflow.com/questions/72695497/how-is-stdcout-implemented)
*   [Is it unethical for nursing school assignments to require students private medical information?](https://academia.stackexchange.com/questions/186236/is-it-unethical-for-nursing-school-assignments-to-require-students-private-medic)
*   [Are chemicals used to ripen fruit in developing countries harmful to the health of consumers?](https://skeptics.stackexchange.com/questions/53458/are-chemicals-used-to-ripen-fruit-in-developing-countries-harmful-to-the-health)
*   [Basis-free, field-independent definition of determinants?](https://math.stackexchange.com/questions/4476061/basis-free-field-independent-definition-of-determinants)
*   [How is energy "stored in an electric field"?](https://physics.stackexchange.com/questions/714872/how-is-energy-stored-in-an-electric-field)
*   [Remove period between Appendix letter and figure number in Appendix captions](https://tex.stackexchange.com/questions/648544/remove-period-between-appendix-letter-and-figure-number-in-appendix-captions)
*   [Are midi-chlorians canon?](https://scifi.stackexchange.com/questions/264587/are-midi-chlorians-canon)
*   [Is this asbestos wiring insulation?](https://diy.stackexchange.com/questions/251464/is-this-asbestos-wiring-insulation)
*   [What would be the least physics breaking way to travel at light speed or faster?](https://worldbuilding.stackexchange.com/questions/231633/what-would-be-the-least-physics-breaking-way-to-travel-at-light-speed-or-faster)
*   [How can I fix a towel bar pulled from the wall?](https://diy.stackexchange.com/questions/251356/how-can-i-fix-a-towel-bar-pulled-from-the-wall)
*   [Best Way to Style Ridiculously Long Hair (for magic purposes)](https://worldbuilding.stackexchange.com/questions/231653/best-way-to-style-ridiculously-long-hair-for-magic-purposes)

[more hot questions](#)

[Question feed](https://stackoverflow.com/feeds/question/46493670 "Feed of this question and its answers")