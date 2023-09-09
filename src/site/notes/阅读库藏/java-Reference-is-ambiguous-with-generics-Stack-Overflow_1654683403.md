---
{"title":"java - Reference is ambiguous with generics - Stack Overflow","url":"https://stackoverflow.com/questions/5361513/reference-is-ambiguous-with-generics","clipped_at":"2022-06-08 18:16:43","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/java-Reference-is-ambiguous-with-generics-Stack-Overflow_1654683403/","dgPassFrontmatter":true}
---


# java - Reference is ambiguous with generics - Stack Overflow

# [Reference is ambiguous with generics](https://stackoverflow.com/questions/5361513/reference-is-ambiguous-with-generics)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 11 years, 2 months ago

Modified [7 years ago](https://stackoverflow.com/questions/5361513/reference-is-ambiguous-with-generics?lastactivity "2015-06-01 17:03:11Z")

Viewed 14k times

37

18

[](https://stackoverflow.com/posts/5361513/timeline)

I'm having quite a tricky case here with generics and method overloading. Check out this example class:

```java
public class Test {
    public <T> void setValue(Parameter<T> parameter, T value) {
    }

    public <T> void setValue(Parameter<T> parameter, Field<T> value) {
    }

    public void test() {
        // This works perfectly. <T> is bound to String
        // ambiguity between setValue(.., String) and setValue(.., Field)
        // is impossible as String and Field are incompatible
        Parameter<String> p1 = getP1();
        Field<String> f1 = getF1();
        setValue(p1, f1);

        // This causes issues. <T> is bound to Object
        // ambiguity between setValue(.., Object) and setValue(.., Field)
        // is possible as Object and Field are compatible
        Parameter<Object> p2 = getP2();
        Field<Object> f2 = getF2();
        setValue(p2, f2);
    }

    private Parameter<String> getP1() {...}
    private Parameter<Object> getP2() {...}

    private Field<String> getF1() {...}
    private Field<Object> getF2() {...}
}
```

The above example compiles perfectly in Eclipse (Java 1.6), but not with the Ant javac command (or with the JDK's javac command), where I get this sort of error message on the second invocation of `setValue`:

> reference to setValue is ambiguous, both method setValue(org.jooq.Parameter,T) in Test and method setValue(org.jooq.Parameter,org.jooq.Field) in Test match

According to the specification and to my understanding of how the Java compiler works, the most specific method should always be chosen: [http://java.sun.com/docs/books/jls/third\_edition/html/expressions.html#20448](http://java.sun.com/docs/books/jls/third_edition/html/expressions.html#20448)

In any case, even if `<T>` is bound to `Object`, which makes both `setValue` methods acceptable candidates for invocation, the one with the `Field` parameter always seems to be more specific. And it works in Eclipse, just not with the JDK's compiler.

**UPDATE**:

Like this, it would work both in Eclipse and with the JDK compiler (with rawtypes warnings, of course). I understand, that the rules specified in [the specs](http://java.sun.com/docs/books/jls/third_edition/html/expressions.html#15.12.2) are quite special, when generics are involved. But I find this rather confusing:

```java
    public <T> void setValue(Parameter<T> parameter, Object value) {
    }

    // Here, it's easy to see that this method is more specific
    public <T> void setValue(Parameter<T> parameter, Field value) {
    }
```

**UPDATE 2**:

Even with generics, I can create this workaround where I avoid the type `<T>` being bound to `Object` at `setValue` invocation time, by adding an additional, unambiguous indirection called `setValue0`. This makes me think that the binding of `T` to `Object` is really what's causing all the trouble here:

```java
    public <T> void setValue(Parameter<T> parameter, T value) {
    }

    public <T> void setValue(Parameter<T> parameter, Field<T> value) {
    }

    public <T> void setValue0(Parameter<T> parameter, Field<T> value) {
        // This call wasn't ambiguous in Java 7
        // It is now ambiguous in Java 8!
        setValue(parameter, value);
    }

    public void test() {
        Parameter<Object> p2 = p2();
        Field<Object> f2 = f2();
        setValue0(p2, f2);
    }
```

Am I misunderstanding something here? Is there a known compiler bug related to this? Or is there a workaround/compiler setting to help me?

## Follow-Up:

For those interested, I have filed a bug report both to Oracle and Eclipse. Oracle has accepted the bug, so far, Eclipse has analysed it and rejected it! It looks as though my intuition is right and this is a bug in `javac`

*   [http://bugs.sun.com/bugdatabase/view\_bug.do?bug\_id=7031404](http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=7031404)
*   [https://bugs.eclipse.org/bugs/show\_bug.cgi?id=340506](https://bugs.eclipse.org/bugs/show_bug.cgi?id=340506)
*   [https://bugs.eclipse.org/bugs/show\_bug.cgi?id=469014](https://bugs.eclipse.org/bugs/show_bug.cgi?id=469014) (a new issue in Eclipse Mars)

[java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [eclipse](https://stackoverflow.com/questions/tagged/eclipse "show questions tagged 'eclipse'") [generics](https://stackoverflow.com/questions/tagged/generics "show questions tagged 'generics'") [javac](https://stackoverflow.com/questions/tagged/javac "show questions tagged 'javac'") [overloading](https://stackoverflow.com/questions/tagged/overloading "show questions tagged 'overloading'")

[Share](https://stackoverflow.com/q/5361513 "Short permalink to this question")

Follow

[edited Jun 1, 2015 at 17:03](https://stackoverflow.com/posts/5361513/revisions "show all edits to this post")

asked Mar 19, 2011 at 10:24

[

![user avatar](/img/user/阅读库藏/assets/1654683403-04a85944492947abce730860ed76a871.jpg)

](https://stackoverflow.com/users/521799/lukas-eder)

[Lukas Eder](https://stackoverflow.com/users/521799/lukas-eder)

197k123123 gold badges648648 silver badges14131413 bronze badges

*   i think, ant is pointing to different compiler which is a not a release candidate.
    
    – [Prince John Wesley](https://stackoverflow.com/users/571189/prince-john-wesley "60,790 reputation")
    
    [Mar 19, 2011 at 11:37](#comment6057712_5361513)
    
*   3
    
    @John: ant uses the JDK's `javac` compiler. It's Eclipse that may have its own...
    
    – [Lukas Eder](https://stackoverflow.com/users/521799/lukas-eder "196,589 reputation")
    
    [Mar 19, 2011 at 13:19](#comment6058343_5361513)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 3 Answers

Sorted by:

Highest score (default) Date modified (newest first) Date created (oldest first)

24

[](https://stackoverflow.com/posts/5365419/timeline)

JDK is right. The 2nd method is not more specific than the 1st. From JLS3#15.12.2.5

"The informal intuition is that one method is more specific than another if **any invocation** handled by the first method could be passed on to the other one without a compile-time type error."

This is clearly not the case here. I emphasized **any invocation**. The property of one method being more specific than the other purely depends on the two methods themselves; it doesn't change per invocation.

Formal analysis on your problem: is m2 more specific than m1?

```java
m1: <R> void setValue(Parameter<R> parameter, R value) 
m2: <V> void setValue(Parameter<V> parameter, Field<V> value) 
```

First, compiler needs to infer R from the initial constraints:

```java
Parameter<V>   <<   Parameter<R>
Field<V>       <<   R
```

The result is `R=V`, per inference rules in 15.12.2.7

Now we substitute `R` and check subtype relations

```java
Parameter<V>   <:   Parameter<V>
Field<V>       <:   V
```

The 2nd line does not hold, per subtyping rules in 4.10.2. So m2 is not more specific than m1.

`V` is not `Object` in this analysis; the analysis considers all possible values of `V`.

I would suggest to use different method names. Overloading is never a necessity.

* * *

This appears to be a significant bug in Eclipse. The spec quite clearly indicates that the type variables are not substituted in this step. Eclipse apparently does type variable substitution first, **then** check method specificity relation.

If such behavior is more "sensible" in some examples, it is not in other examples. Say,

```java
m1: <T extends Object> void check(List<T> list, T obj) { print("1"); }
m2: <T extends Number> void check(List<T> list, T num) { print("2"); }

void test()
    check( new ArrayList<Integer>(), new Integer(0) );
```

"Intuitively", and formally per spec, m2 is more specific than m1, and the test prints "2". However, if substitution `T=Integer` is done first, the two methods become identical!

* * *

**for Update 2**

```java
m1: <R> void setValue(Parameter<R> parameter, R value) 
m2: <V> void setValue(Parameter<V> parameter, Field<V> value) 

m3: <T> void setValue2(Parameter<T> parameter, Field<T> value)
s4:             setValue(parameter, value)
```

Here, m1 is not applicable for method invocation s4, so m2 is the only choice.

Per 15.12.2.2, to see if m1 is applicable for s4, first, type inference is carried out, to the conclusion that R=T; then we check `Ai :< Si`, which leads to `Field<T> <: T`, which is false.

This is consistent with the previous analysis - if m1 is applicable to s4, then any invocation handled by m2 (essentially same as s4) can be handled by m1, which means m2 would be more specific than m1, which is false.

**in a parameterized type**

Consider the following code

```java
class PF<T>
{
    public void setValue(Parameter<T> parameter, T value) {
    }

    public void setValue(Parameter<T> parameter, Field<T> value) {
    }
}

void test()

    PF<Object> pf2 = null;

    Parameter<Object> p2 = getP2();
    Field<Object> f2 = getF2();

    pf2.setValue(p2,f2);
```

This compiles without problem. Per 4.5.2, the types of the methods in `PF<Object>` are methods in `PF<T>` with substitution `T=Object`. That is, the methods of `pf2` are

```java
    public void setValue(Parameter<Object> parameter, Object value) 

    public void setValue(Parameter<Object> parameter, Field<Object> value) 
```

The 2nd method is more specific than the 1st.

[Share](https://stackoverflow.com/a/5365419 "Short permalink to this answer")

Follow

[edited Mar 21, 2011 at 10:12](https://stackoverflow.com/posts/5365419/revisions "show all edits to this post")

answered Mar 19, 2011 at 22:41

[

![user avatar](/img/user/阅读库藏/assets/1654683403-22abc1e304a4fd47d456908b8237aaba.png)

](https://stackoverflow.com/users/218978/irreputable)

[irreputable](https://stackoverflow.com/users/218978/irreputable)

43.8k99 gold badges6363 silver badges9292 bronze badges

*   1
    
    @irreputable, you are right about avoiding overloading in this case. Since the situation is too complex for me to understand, that's the safest way to go. I don't understand your annoyance, though. Eclipse is by IBM, javac by Sun/Oracle. I'm sure the spec itself is right, I never doubted that. But javac is just another implementation of that spec to me. And even if your argument makes sense, it could've been the other way round. After all, you discovered a javac bug yourself, as I can see from your stackoverflow questions...
    
    – [Lukas Eder](https://stackoverflow.com/users/521799/lukas-eder "196,589 reputation")
    
    [Mar 20, 2011 at 1:32](#comment6063653_5365419)
    
*   @irreputable, there's one more thing worrying me. If `V` being bound to `Object` didn't matter as you say, then why is **UPDATE 2** in my question compilable?
    
    – [Lukas Eder](https://stackoverflow.com/users/521799/lukas-eder "196,589 reputation")
    
    [Mar 20, 2011 at 1:35](#comment6063674_5365419)
    
*   @lukas-eder see my addition to your update 2. I agree such detailed study is not worth the time, unless for fun. About javac, it is not _just another implementation_. It's developed side by side with the spec, by the same people; some features were implemented in javac first, then copied to spec. Other compilers don't have such luxury.
    
    – [irreputable](https://stackoverflow.com/users/218978/irreputable "43,835 reputation")
    
    [Mar 20, 2011 at 2:42](#comment6064004_5365419)
    
*   2
    
    @irreputable: Your addition makes sense. All of this is just generic nightmare. I still think that Eclipse's compiler behaves "better" in this case, because it may include one additional (wrong according to the specs) rule of disambiguation, taking into account that with `Object` as bound for `T` there is ambiguity according to the specs. But that's my very informal and intuitive opinion, as a compiler-anti-expert :-).
    
    – [Lukas Eder](https://stackoverflow.com/users/521799/lukas-eder "196,589 reputation")
    
    [Mar 20, 2011 at 10:02](#comment6065960_5365419)
    
*   1
    
    I still haven't seen a fully consistent explanation that's fully in line with JLS3, but the situation is much clearer in JLS8 and at this level javac and ecj agree on reporting ambiguity.
    
    – [Stephan Herrmann](https://stackoverflow.com/users/4611488/stephan-herrmann "7,603 reputation")
    
    [Jun 4, 2015 at 14:26](#comment49356786_5365419)
    

[Show **7** more comments](# "Expand to show all comments on this post")

0

[](https://stackoverflow.com/posts/5361589/timeline)

My guess is that the compiler is doing an method overloading resolution as per [JLS, Section 15.12.2.5](http://java.sun.com/docs/books/jls/third_edition/html/expressions.html#301183).

For this Section, the compiler uses [**strong subtyping**](http://java.sun.com/docs/books/jls/third_edition/html/expressions.html#301185) (thus not allowing any unchecked conversion), so, `T value` becomes `Object value` and `Field<T> value` becomes `Field<Object> value`. The following rules will apply:

> The method _m_ is applicable by subtyping if and only if both of the following conditions hold:
> 
> ```java
> * For 1in, either:
>       o Ai is a subtype (§4.10) of Si (Ai <: Si) or
>       o Ai is convertible to some type *Ci* by unchecked conversion
> ```
> 
> (§5.1.9), and Ci <: Si. \* If m is a generic method as described above then Ul <: Bl\[R1 = U1, ..., Rp = Up\], 1lp.

(Refer to bullet 2). Since `Field<Object>` is a subtype of `Object` then the most specific method is found. Field `f2` matches both methods of yours (because of bullet 2 above) and makes it ambiguous.

For `String` and `Field<String>`, there is no subtype relationship between the two.

PS. This is my understanding of things, don't quote it as kosher.

[Share](https://stackoverflow.com/a/5361589 "Short permalink to this answer")

Follow

[edited Mar 19, 2011 at 11:40](https://stackoverflow.com/posts/5361589/revisions "show all edits to this post")

answered Mar 19, 2011 at 10:41

[

![user avatar](/img/user/阅读库藏/assets/1654683403-faaeb60b3b1ce7715406ea8d37ecda2b.png)

](https://stackoverflow.com/users/251173/buhake-sindi)

[Buhake Sindi](https://stackoverflow.com/users/251173/buhake-sindi)

85.6k2727 gold badges164164 silver badges223223 bronze badges

*   1
    
    Calls to overloaded methods are resolved at compile time, aren't they? Also, why does the `String` case work?
    
    – [Marcelo Cantos](https://stackoverflow.com/users/9990/marcelo-cantos "174,473 reputation")
    
    [Mar 19, 2011 at 10:42](#comment6057369_5361589)
    
*   Then why do I have `T` in the compiler error message, if `T` were already erased? Besides, `Object` and `Field` are still distinct types, even after type erasure
    
    – [Lukas Eder](https://stackoverflow.com/users/521799/lukas-eder "196,589 reputation")
    
    [Mar 19, 2011 at 10:45](#comment6057395_5361589)
    
*   Thanks, my first answer didn't make sense. I hope this helps.
    
    – [Buhake Sindi](https://stackoverflow.com/users/251173/buhake-sindi "85,574 reputation")
    
    [Mar 19, 2011 at 11:40](#comment6057730_5361589)
    
*   With unchecked conversion we would see a compile time warning, no? I think the matter is more complicated than that.
    
    – [Georgy Bolyuba](https://stackoverflow.com/users/4052/georgy-bolyuba "8,145 reputation")
    
    [Mar 19, 2011 at 11:51](#comment6057791_5361589)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/5361948/timeline)

**Edit**: This answer is wrong. Take a look at accepted answer.

I think the issue comes down this: compiler does not see the type of f2 (i.e. Field) and the inferred type of formal parameter (i.e. Field -> Field) as the same type.

In other words, it looks like type of f2 (Field) is considered to be a subtype of the type of formal parameter Field (Field). Since Field is at the same type a subtype of Object, compiler cannot pick one method over another.

**Edit**: Let me expand my statement a bit

Both methods are [applicable](http://java.sun.com/docs/books/jls/third_edition/html/expressions.html#15.12.2.1) and it looks like the [Phase 1: Identify Matching Arity Methods Applicable by Subtyping](http://java.sun.com/docs/books/jls/third_edition/html/expressions.html#15.12.2.2) is used to decide which method to call and than rules from [Choosing the Most Specific Method](http://java.sun.com/docs/books/jls/third_edition/html/expressions.html#301183) applied, but failed for some reason to pick second method over first one.

_Phase 1_ section uses this notation: `X <: S` (X is subtype of S). Based on my understanding of `<:`, `X <: X` is a valid expression, i.e. the `<:` is not strict and includes the type itself (X is subtype of X) in this context. This explains the result of Phase 1: both methods are picked as candidates, since `Field<Object> <: Object` and `Field<Object> <: Field<Object>`.

_Choosing the Most Specific Method_ section uses same notation to say that one method is more specific than another. The interesting part the paragraph that starts with "One fixed-arity member method named m is more specific than another member...". It has, among other things:

> For all j from 1 to n, Tj <: Sj.

This makes me think that in our case second method **must** be chosen over the first one, because following holds:

*   `Parameter<Object> <: Parameter<Object>`
*   `Field<Object> <: Object`

while the other way around does not hold due to `Object <: Field<Object>` being false (Object is not a subtype of Field).

Note: In case of String examples, Phase 1 will simply pick the only method applicable: the second one.

So, to answer your questions: I think this is a bug in compiler implementation. Eclipse has it is own incremental compiler which does not have this bug it seems.

[Share](https://stackoverflow.com/a/5361948 "Short permalink to this answer")

Follow

[edited Mar 20, 2011 at 21:05](https://stackoverflow.com/posts/5361948/revisions "show all edits to this post")

answered Mar 19, 2011 at 12:06

[

![user avatar](/img/user/阅读库藏/assets/1654683403-d145160c02c5fc872d750e5100342f4d.jpg)

](https://stackoverflow.com/users/4052/georgy-bolyuba)

[Georgy Bolyuba](https://stackoverflow.com/users/4052/georgy-bolyuba)

8,14577 gold badges2727 silver badges3535 bronze badges

*   Could you explain with an example
    
    – [Deepak](https://stackoverflow.com/users/511125/deepak "2,074 reputation")
    
    [Mar 19, 2011 at 15:25](#comment6059186_5361948)
    
*   With an example of what exactly?
    
    – [Georgy Bolyuba](https://stackoverflow.com/users/4052/georgy-bolyuba "8,145 reputation")
    
    [Mar 19, 2011 at 17:54](#comment6060205_5361948)
    
*   2
    
    @GeorgyBolyuba: if your answer is wrong: why don't you delete it?
    
    – [LisaMM](https://stackoverflow.com/users/3580458/lisamm "665 reputation")
    
    [Jul 29, 2015 at 8:19](#comment51330169_5361948)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

  

## Your Answer

### Sign up or [log in](https://stackoverflow.com/users/login?ssrc=question_page&returnurl=https%3a%2f%2fstackoverflow.com%2fquestions%2f5361513%2freference-is-ambiguous-with-generics%23new-answer)

Sign up using Google

Sign up using Facebook

Sign up using Email and Password

 

### Post as a guest

Name

Email

Required, but never shown

Post Your Answer

By clicking “Post Your Answer”, you agree to our [terms of service](https://stackoverflow.com/legal/terms-of-service/public), [privacy policy](https://stackoverflow.com/legal/privacy-policy) and [cookie policy](https://stackoverflow.com/legal/cookie-policy)

## Not the answer you're looking for? Browse other questions tagged [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [eclipse](https://stackoverflow.com/questions/tagged/eclipse "show questions tagged 'eclipse'") [generics](https://stackoverflow.com/questions/tagged/generics "show questions tagged 'generics'") [javac](https://stackoverflow.com/questions/tagged/javac "show questions tagged 'javac'") [overloading](https://stackoverflow.com/questions/tagged/overloading "show questions tagged 'overloading'") or [ask your own question](https://stackoverflow.com/questions/ask).

The Overflow Blog

*   [Remote work is killing big offices. Cities must change to survive](https://stackoverflow.blog/2022/06/06/remote-work-is-killing-big-offices-cities-must-change-to-survive/?cb=1)
    
*   [On the quantum internet, data doesn’t stream; it teleports](https://stackoverflow.blog/2022/06/07/on-the-quantum-internet-data-doesnt-steam-it-teleports/?cb=1)
    

Featured on Meta

*   [Announcing the arrival of Valued Associate #1214: Dalmarus](https://meta.stackexchange.com/questions/378862/announcing-the-arrival-of-valued-associate-1214-dalmarus?cb=1)
    
*   [Improvements to site status and incident communication](https://meta.stackexchange.com/questions/378941/improvements-to-site-status-and-incident-communication?cb=1)
    
*   [Collectives Update: Introducing Bulletins](https://meta.stackoverflow.com/questions/418333/collectives-update-introducing-bulletins?cb=1)
    
*   [The \[comma\] tag is being burninated](https://meta.stackoverflow.com/questions/405831/the-comma-tag-is-being-burninated?cb=1)
    
*   [Ask Wizard Test Results and Next Steps](https://meta.stackoverflow.com/questions/418491/ask-wizard-test-results-and-next-steps?cb=1)
    

#### Linked

[

4

](https://stackoverflow.com/q/7943600?lq=1 "Question score (upvotes - downvotes)")[Reference to method is ambiguous, with generics, odd behaviour](https://stackoverflow.com/questions/7943600/reference-to-method-is-ambiguous-with-generics-odd-behaviour?noredirect=1&lq=1)

[

\-2

](https://stackoverflow.com/q/26318691?lq=1 "Question score (upvotes - downvotes)")[How can I make a overhead a method with the parameter HashMap?](https://stackoverflow.com/questions/26318691/how-can-i-make-a-overhead-a-method-with-the-parameter-hashmap?noredirect=1&lq=1)

[

26

](https://stackoverflow.com/q/28466925?lq=1 "Question score (upvotes - downvotes)")[Java type inference: reference is ambiguous in Java 8, but not Java 7](https://stackoverflow.com/questions/28466925/java-type-inference-reference-is-ambiguous-in-java-8-but-not-java-7?noredirect=1&lq=1)

[

46

](https://stackoverflow.com/q/10642241?lq=1 "Question score (upvotes - downvotes)")[Ambiguous overloaded java methods with generics and varargs](https://stackoverflow.com/questions/10642241/ambiguous-overloaded-java-methods-with-generics-and-varargs?noredirect=1&lq=1)

[

6

](https://stackoverflow.com/q/8601307?lq=1 "Question score (upvotes - downvotes)")[Widening and boxing with Java](https://stackoverflow.com/questions/8601307/widening-and-boxing-with-java?noredirect=1&lq=1)

[

4

](https://stackoverflow.com/q/11500385?lq=1 "Question score (upvotes - downvotes)")[How does the JLS specify that wildcards cannot be formally used within methods?](https://stackoverflow.com/questions/11500385/how-does-the-jls-specify-that-wildcards-cannot-be-formally-used-within-methods?noredirect=1&lq=1)

[

2

](https://stackoverflow.com/q/5692466?lq=1 "Question score (upvotes - downvotes)")[Calling ambiguously overloaded constructor in Java](https://stackoverflow.com/questions/5692466/calling-ambiguously-overloaded-constructor-in-java?noredirect=1&lq=1)

[

4

](https://stackoverflow.com/q/6573171?lq=1 "Question score (upvotes - downvotes)")[Why are there errors in released Java source code?](https://stackoverflow.com/questions/6573171/why-are-there-errors-in-released-java-source-code?noredirect=1&lq=1)

[

5

](https://stackoverflow.com/q/64759424?lq=1 "Question score (upvotes - downvotes)")[kotlin overload resolution ambiguity when generic type is bound to Any](https://stackoverflow.com/questions/64759424/kotlin-overload-resolution-ambiguity-when-generic-type-is-bound-to-any?noredirect=1&lq=1)

[

2

](https://stackoverflow.com/q/7747052?lq=1 "Question score (upvotes - downvotes)")[Generics - Eclipse compiler issues a warning, javac gives an error](https://stackoverflow.com/questions/7747052/generics-eclipse-compiler-issues-a-warning-javac-gives-an-error?noredirect=1&lq=1)

[See more linked questions](https://stackoverflow.com/questions/linked/5361513?lq=1)

#### Related

[

7427

](https://stackoverflow.com/q/40480?rq=1 "Question score (upvotes - downvotes)")[Is Java "pass-by-reference" or "pass-by-value"?](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value?rq=1)

[

2746

](https://stackoverflow.com/q/574594?rq=1 "Question score (upvotes - downvotes)")[How can I create an executable JAR with dependencies using Maven?](https://stackoverflow.com/questions/574594/how-can-i-create-an-executable-jar-with-dependencies-using-maven?rq=1)

[

856

](https://stackoverflow.com/q/2745265?rq=1 "Question score (upvotes - downvotes)")[Is List<Dog> a subclass of List<Animal>? Why are Java generics not implicitly polymorphic?](https://stackoverflow.com/questions/2745265/is-listdog-a-subclass-of-listanimal-why-are-java-generics-not-implicitly-po?rq=1)

[

9

](https://stackoverflow.com/q/3452859?rq=1 "Question score (upvotes - downvotes)")[Java generics code compiles with javac, fails with Eclipse Helios](https://stackoverflow.com/questions/3452859/java-generics-code-compiles-with-javac-fails-with-eclipse-helios?rq=1)

[

23

](https://stackoverflow.com/q/4829576?rq=1 "Question score (upvotes - downvotes)")[javac error: inconvertible types with generics?](https://stackoverflow.com/questions/4829576/javac-error-inconvertible-types-with-generics?rq=1)

[

3

](https://stackoverflow.com/q/7826671?rq=1 "Question score (upvotes - downvotes)")[Java generics Eclipse compiler bug?](https://stackoverflow.com/questions/7826671/java-generics-eclipse-compiler-bug?rq=1)

[

3

](https://stackoverflow.com/q/20201731?rq=1 "Question score (upvotes - downvotes)")[Eclipse Generics Issue - Workaround?](https://stackoverflow.com/questions/20201731/eclipse-generics-issue-workaround?rq=1)

[

15

](https://stackoverflow.com/q/26222510?rq=1 "Question score (upvotes - downvotes)")[Ambiguous overload in Java8 - is ECJ or javac right?](https://stackoverflow.com/questions/26222510/ambiguous-overload-in-java8-is-ecj-or-javac-right?rq=1)

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [What are those stars that cross the galactic center?](https://astronomy.stackexchange.com/questions/49524/what-are-those-stars-that-cross-the-galactic-center)
*   [Strange dynamic marking 'fmo' in Clementi sonata](https://music.stackexchange.com/questions/123264/strange-dynamic-marking-fmo-in-clementi-sonata)
*   [Ubuntu 20.04 download link](https://askubuntu.com/questions/1412658/ubuntu-20-04-download-link)
*   [Permanently change symbol of itemize](https://tex.stackexchange.com/questions/647006/permanently-change-symbol-of-itemize)
*   [Is there enough support in the UK parliament for a no confidence vote in the PM?](https://politics.stackexchange.com/questions/73478/is-there-enough-support-in-the-uk-parliament-for-a-no-confidence-vote-in-the-pm)
*   [Why do authoritarian regimes bother keeping opposition leaders alive in prison?](https://politics.stackexchange.com/questions/73451/why-do-authoritarian-regimes-bother-keeping-opposition-leaders-alive-in-prison)
*   [What is the penalty for a bad-acting juror?](https://law.stackexchange.com/questions/80860/what-is-the-penalty-for-a-bad-acting-juror)
*   [Photo Competition 2022-06-06: Reflections](https://photo.stackexchange.com/questions/129502/photo-competition-2022-06-06-reflections)
*   [What number should the first person announce?](https://puzzling.stackexchange.com/questions/116520/what-number-should-the-first-person-announce)
*   [Is it possible to have a Newtonian universe but everything else behave the same way as this universe? What adjustments to physics is necessary?](https://worldbuilding.stackexchange.com/questions/230991/is-it-possible-to-have-a-newtonian-universe-but-everything-else-behave-the-same)
*   [Does a Car Keep Moving If Neither Accelerator, Nor Brake Is Pressed?](https://mechanics.stackexchange.com/questions/88927/does-a-car-keep-moving-if-neither-accelerator-nor-brake-is-pressed)
*   [Can I see how a company is using its debt?](https://money.stackexchange.com/questions/151215/can-i-see-how-a-company-is-using-its-debt)
*   [Extra spacing on the left of operator?](https://tex.stackexchange.com/questions/646942/extra-spacing-on-the-left-of-operator)
*   [Is printing money any different from taxing people?](https://politics.stackexchange.com/questions/73498/is-printing-money-any-different-from-taxing-people)
*   [Feeling like a fraud at a startup I joined as an intern](https://workplace.stackexchange.com/questions/185410/feeling-like-a-fraud-at-a-startup-i-joined-as-an-intern)
*   [How can I find out the number of teeth on each cog for cassettes and chain rings?](https://bicycles.stackexchange.com/questions/84217/how-can-i-find-out-the-number-of-teeth-on-each-cog-for-cassettes-and-chain-rings)
*   [Why is Sam unsure about whether or not he has taken out the Elf-rope before in chapter 12 of Two Towers?](https://scifi.stackexchange.com/questions/264058/why-is-sam-unsure-about-whether-or-not-he-has-taken-out-the-elf-rope-before-in-c)
*   [Is making all Opportunity Attacks have disadvantage game breaking in any way?](https://rpg.stackexchange.com/questions/198924/is-making-all-opportunity-attacks-have-disadvantage-game-breaking-in-any-way)
*   [Are there modern languages without standardized spelling? If not, why?](https://linguistics.stackexchange.com/questions/44594/are-there-modern-languages-without-standardized-spelling-if-not-why)
*   [I just bought a house and I see a gas line, 110v and 220v outlet in the laundry room. Does this mean I can install either a gas or electric dryer?](https://diy.stackexchange.com/questions/250595/i-just-bought-a-house-and-i-see-a-gas-line-110v-and-220v-outlet-in-the-laundry)
*   [Number of registrants for a virtual talk I organized turned out to be small, should I notify the speaker beforehand?](https://academia.stackexchange.com/questions/185926/number-of-registrants-for-a-virtual-talk-i-organized-turned-out-to-be-small-sho)
*   [What is non-networked Unix domain socket?](https://superuser.com/questions/1725107/what-is-non-networked-unix-domain-socket)
*   [Inner sill corrosion implications in Rover 45 2003](https://mechanics.stackexchange.com/questions/88944/inner-sill-corrosion-implications-in-rover-45-2003)
*   [What is \`/dev/sda0\`? Is it a standard thing?](https://unix.stackexchange.com/questions/705252/what-is-dev-sda0-is-it-a-standard-thing)

[Question feed](https://stackoverflow.com/feeds/question/5361513 "Feed of this question and its answers")