---
{"title":"list - Java 8 stream reverse order - Stack Overflow","url":"https://stackoverflow.com/questions/24010109/java-8-stream-reverse-order","clipped_at":"2022-07-22 06:31:35","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/list-Java-8-stream-reverse-order-Stack-Overflow_1658442695/","dgPassFrontmatter":true}
---


# list - Java 8 stream reverse order - Stack Overflow

# [Java 8 stream reverse order](https://stackoverflow.com/questions/24010109/java-8-stream-reverse-order)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 8 years, 1 month ago

Modified [6 months ago](https://stackoverflow.com/questions/24010109/java-8-stream-reverse-order?lastactivity "2022-01-10 17:11:50Z")

Viewed 268k times

209

32

[](https://stackoverflow.com/posts/24010109/timeline)

General question: What's the proper way to reverse a stream? Assuming that we don't know what type of elements that stream consists of, what's the generic way to reverse any stream?

Specific question:

`IntStream` provides range method to generate Integers in specific range `IntStream.range(-range, 0)`, now that I want to reverse it switching range from 0 to negative won't work, also I can't use `Integer::compare`

```java
List<Integer> list = Arrays.asList(1,2,3,4);
list.stream().sorted(Integer::compare).forEach(System.out::println);
```

with `IntStream` I'll get this compiler error

> Error:(191, 0) ajc: The method `sorted()` in the type `IntStream` is not applicable for the arguments (`Integer::compare`)

what am I missing here?

[java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [list](https://stackoverflow.com/questions/tagged/list "show questions tagged 'list'") [sorting](https://stackoverflow.com/questions/tagged/sorting "show questions tagged 'sorting'") [java-8](https://stackoverflow.com/questions/tagged/java-8 "show questions tagged 'java-8'") [java-stream](https://stackoverflow.com/questions/tagged/java-stream "show questions tagged 'java-stream'")

[Share](https://stackoverflow.com/q/24010109 "Short permalink to this question")

Follow

[edited Mar 18, 2020 at 16:52](https://stackoverflow.com/posts/24010109/revisions "show all edits to this post")

[

![user avatar](/img/user/阅读库藏/assets/1658442695-065a581e7ccb20d06b62e1496f445b7a.jpg)

](https://stackoverflow.com/users/10279528/nilanka-manoj)

[Nilanka Manoj](https://stackoverflow.com/users/10279528/nilanka-manoj)

3,36033 gold badges1313 silver badges4444 bronze badges

asked Jun 3, 2014 at 8:09

[

![user avatar](/img/user/阅读库藏/assets/1658442695-82e6ad47632348bc7d1c4ac5e8def045.jpg)

](https://stackoverflow.com/users/654799/vach)

[vach](https://stackoverflow.com/users/654799/vach)

9,6571111 gold badges6060 silver badges9797 bronze badges

*   2
    
    An `IntStream` has no `.sorted(Comparator)` method; you have to go through a `Stream<Integer>` first and reverse there before yielding an `IntStream`
    
    – [fge](https://stackoverflow.com/users/1093528/fge "115,306 reputation")
    
    [Jun 3, 2014 at 8:23](#comment37006195_24010109)
    
*   5
    
    To generate an `IntStream.range(0, n)` in reverse order, do something like `map(i -> n - i - 1)`. No need to do boxing and sorting.
    
    – [Stuart Marks](https://stackoverflow.com/users/1441122/stuart-marks "121,504 reputation")
    
    [Jun 3, 2014 at 8:39](#comment37006672_24010109)
    
*   3
    
    Your gerneral question and your specific question read like two completele different questions to me. The general speaks of reversing the _stream_, while the specific speaks of ordering numbers in descending order. If the stream produces the numbers in an unordered manner like `1, 3, 2`, what is your expected outcome? Do you want the reversed stream like `2, 3, 1` or the sorted stream like `3, 2, 1`?
    
    – [chiccodoro](https://stackoverflow.com/users/55787/chiccodoro "14,089 reputation")
    
    [Jun 3, 2014 at 8:54](#comment37007119_24010109)
    
*   9
    
    You can't reverse a stream in general - for example a stream may be inifinite.
    
    – [assylias](https://stackoverflow.com/users/829571/assylias "311,654 reputation")
    
    [Jun 3, 2014 at 9:01](#comment37007343_24010109)
    
*   1
    
    You may want to rephrase the question as "Iterate a collection in reverse order in Java 8 way". Answer may be beyond streams. Below answer from @venkata-raju solves the problem, but takes extra space. I'm still waiting to see a good answer on this question.
    
    – [Manu Manjunath](https://stackoverflow.com/users/495598/manu-manjunath "5,981 reputation")
    
    [Dec 10, 2015 at 12:56](#comment56149526_24010109)
    

[Show **5** more comments](# "Expand to show all comments on this post")

## 30 Answers

Sorted by:

Trending sort available

Highest score (default) Trending (recent votes count more) Date modified (newest first) Date created (oldest first)

107

[](https://stackoverflow.com/posts/24011264/timeline)

For the specific question of generating a reverse `IntStream`, try something like this:

```java
static IntStream revRange(int from, int to) {
    return IntStream.range(from, to)
                    .map(i -> to - i + from - 1);
}
```

This avoids boxing and sorting.

For the general question of how to reverse a stream of any type, I don't know of there's a "proper" way. There are a couple ways I can think of. Both end up storing the stream elements. I don't know of a way to reverse a stream without storing the elements.

This first way stores the elements into an array and reads them out to a stream in reverse order. Note that since we don't know the runtime type of the stream elements, we can't type the array properly, requiring an unchecked cast.

```java
@SuppressWarnings("unchecked")
static <T> Stream<T> reverse(Stream<T> input) {
    Object[] temp = input.toArray();
    return (Stream<T>) IntStream.range(0, temp.length)
                                .mapToObj(i -> temp[temp.length - i - 1]);
}
```

Another technique uses collectors to accumulate the items into a reversed list. This does lots of insertions at the front of `ArrayList` objects, so there's lots of copying going on.

```java
Stream<T> input = ... ;
List<T> output =
    input.collect(ArrayList::new,
                  (list, e) -> list.add(0, e),
                  (list1, list2) -> list1.addAll(0, list2));
```

It's probably possible to write a much more efficient reversing collector using some kind of customized data structure.

**UPDATE 2016-01-29**

Since this question has gotten a bit of attention recently, I figure I should update my answer to solve the problem with inserting at the front of `ArrayList`. This will be horribly inefficient with a large number of elements, requiring O(N^2) copying.

It's preferable to use an `ArrayDeque` instead, which efficiently supports insertion at the front. A small wrinkle is that we can't use the three-arg form of `Stream.collect()`; it requires the contents of the second arg be merged into the first arg, and there's no "add-all-at-front" bulk operation on `Deque`. Instead, we use `addAll()` to append the contents of the first arg to the end of the second, and then we return the second. This requires using the `Collector.of()` factory method.

The complete code is this:

```java
Deque<String> output =
    input.collect(Collector.of(
        ArrayDeque::new,
        (deq, t) -> deq.addFirst(t),
        (d1, d2) -> { d2.addAll(d1); return d2; }));
```

The result is a `Deque` instead of a `List`, but that shouldn't be much of an issue, as it can easily be iterated or streamed in the now-reversed order.

[Share](https://stackoverflow.com/a/24011264 "Short permalink to this answer")

Follow

[edited Jan 29, 2016 at 18:31](https://stackoverflow.com/posts/24011264/revisions "show all edits to this post")

answered Jun 3, 2014 at 9:13

[

![user avatar](/img/user/阅读库藏/assets/1658442695-da31f69410ed9a75870b562c903e6db6.jpg)

](https://stackoverflow.com/users/1441122/stuart-marks)

[Stuart Marks](https://stackoverflow.com/users/1441122/stuart-marks)

122k3535 gold badges194194 silver badges254254 bronze badges

*   18
    
    Alternatively: `IntStream.iterate(to-1, i->i-1).limit(to-from)`
    
    – [Holger](https://stackoverflow.com/users/2711488/holger "269,089 reputation")
    
    [Jun 3, 2014 at 11:41](#comment37013255_24011264)
    
*   3
    
    @Holger Unfortunately, that solution doesn't handle overflow correctly.
    
    – [Brandon](https://stackoverflow.com/users/1237044/brandon "2,298 reputation")
    
    [Jan 28, 2016 at 23:51](#comment57871761_24011264)
    
*   5
    
    @Brandon Mintern: indeed, you would have to use `.limit(endExcl-(long)startIncl)` instead, but for such large streams, it is very discouraged anyway as it’s much less efficient than the `range` based solution. At the time I wrote the comment, I wasn’t aware about the efficiency difference.
    
    – [Holger](https://stackoverflow.com/users/2711488/holger "269,089 reputation")
    
    [Jan 29, 2016 at 9:48](#comment57884827_24011264)
    
*   Or `IntStream.rangeClosed(1, to - from).map(i -> to-i)` (see [bjmi's comment](https://stackoverflow.com/questions/24010109/java-8-stream-reverse-order/36192274#comment122436338_24010109), too).
    
    – [greybeard](https://stackoverflow.com/users/3789665/greybeard "2,102 reputation")
    
    [Jan 10 at 10:02](#comment124894454_24011264)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

65

[](https://stackoverflow.com/posts/36192274/timeline)

Elegant solution

```java
List<Integer> list = Arrays.asList(1,2,3,4);
list.stream()
    .sorted(Collections.reverseOrder()) // Method on Stream<Integer>
    .forEach(System.out::println);
```

[Share](https://stackoverflow.com/a/36192274 "Short permalink to this answer")

Follow

[edited Jan 10 at 17:11](https://stackoverflow.com/posts/36192274/revisions "show all edits to this post")

[

![user avatar](/img/user/阅读库藏/assets/1658442695-08ac675535b0cfa1ec77d202778668b9.png)

](https://stackoverflow.com/users/11133060/tjmnkrajyej)

[tjmnkrajyej](https://stackoverflow.com/users/11133060/tjmnkrajyej)

911 silver badge55 bronze badges

answered Mar 24, 2016 at 2:32

[

![user avatar](/img/user/阅读库藏/assets/1658442695-f0403b034658760e0a46ef277fc563a0.png)

](https://stackoverflow.com/users/3316017/kishan-b)

[Kishan B](https://stackoverflow.com/users/3316017/kishan-b)

3,80511 gold badge1818 silver badges1010 bronze badges

*   10
    
    It is elegant but not fully working as it seems as elements of the list are required to be `Comparable`...
    
    – [Krzysztof Wolny](https://stackoverflow.com/users/209502/krzysztof-wolny "9,790 reputation")
    
    [Aug 4, 2016 at 15:11](#comment64913726_36192274)
    
*   81
    
    That's assuming we want elements to be _sorted_ in reverse order. The question is about reversing the order of a stream.
    
    – [Dillon Ryan Redding](https://stackoverflow.com/users/1560086/dillon-ryan-redding "1,103 reputation")
    
    [Nov 4, 2018 at 14:24](#comment93175244_36192274)
    
*   @KrzysztofWolny you can pass a comparator function to reverseOrder(), so that shouldn't be too much of a bother
    
    – [Dávid Leblay](https://stackoverflow.com/users/9409790/d%c3%a1vid-leblay "140 reputation")
    
    [Sep 21, 2020 at 12:45](#comment113158347_36192274)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

49

[](https://stackoverflow.com/posts/24183443/timeline)

General Question:

Stream does not store any elements.

So iterating elements in the reverse order is not possible without storing the elements in some intermediate collection.

```java
Stream.of("1", "2", "20", "3")
      .collect(Collectors.toCollection(ArrayDeque::new)) // or LinkedList
      .descendingIterator()
      .forEachRemaining(System.out::println);
```

Update: Changed LinkedList to ArrayDeque (better) [see here for details](https://stackoverflow.com/questions/6163166/why-is-arraydeque-better-than-linkedlist)

Prints:

```java
3
20
2
1
```

By the way, using `sort` method is not correct as it sorts, NOT reverses (assuming stream may have unordered elements)

Specific Question:

I found this simple, easier and intuitive(Copied @Holger [comment](https://stackoverflow.com/questions/24010109/java-8-stream-reverse-order/24183443#comment37013255_24011264))

```java
IntStream.iterate(to - 1, i -> i - 1).limit(to - from)
```

[Share](https://stackoverflow.com/a/24183443 "Short permalink to this answer")

Follow

[edited Oct 3, 2019 at 13:07](https://stackoverflow.com/posts/24183443/revisions "show all edits to this post")

answered Jun 12, 2014 at 11:40

[

![user avatar](/img/user/阅读库藏/assets/1658442695-622a304cdaafcc0eb44a4e330dad84ff.jpg)

](https://stackoverflow.com/users/1812434/venkata-raju)

[Venkata Raju](https://stackoverflow.com/users/1812434/venkata-raju)

4,47633 gold badges2626 silver badges3434 bronze badges

*   3
    
    Some stream operation, such as `sorted` and `distinct` actually store an intermediate result. See the [package API docs](http://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html#StreamOps) for some information about that.
    
    – [Lii](https://stackoverflow.com/users/452775/lii "10,886 reputation")
    
    [Sep 25, 2014 at 9:08](#comment40780791_24183443)
    
*   @Lii I see `No storage` in the same page. Even it stores we can't get access to that storage (so `No storage` is fine i guess)
    
    – [Venkata Raju](https://stackoverflow.com/users/1812434/venkata-raju "4,476 reputation")
    
    [Oct 16, 2015 at 14:13](#comment54152693_24183443)
    
*   Good answer. But since extra space is used, it many not be a great idea for programmers to use your approach on very large collection.
    
    – [Manu Manjunath](https://stackoverflow.com/users/495598/manu-manjunath "5,981 reputation")
    
    [Dec 10, 2015 at 12:54](#comment56149432_24183443)
    
*   I like the simplicity of the solution from an understandability perspective and leveraging an existing method on an existing data structure...the solution previous with a map implementation is harder to understand but for sure Manu is right, for large collections I would not use this intuitive, and would opt for the map one above.
    
    – [Beezer](https://stackoverflow.com/users/3074057/beezer "994 reputation")
    
    [Dec 2, 2016 at 4:56](#comment69061525_24183443)
    
*   This is one of the few correct answers here. Most of the other ones don't actually reverse the stream, they try to avoid doing it somehow (which only works under peculiar sets of circumstances under which you normally wouldn't need to reverse in the first place). If you try to reverse a stream that doesn't fit into memory you're doing it wrong anyway. Dump it into a DB and get a reversed stream using regular SQL.
    
    – [Cubic](https://stackoverflow.com/users/938694/cubic "14,102 reputation")
    
    [Oct 3, 2019 at 13:13](#comment102815628_24183443)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

39

[](https://stackoverflow.com/posts/34621495/timeline)

Many of the solutions here sort or reverse the `IntStream`, but that unnecessarily requires intermediate storage. [Stuart Marks's solution](https://stackoverflow.com/a/24011264/2711488) is the way to go:

```java
static IntStream revRange(int from, int to) {
    return IntStream.range(from, to).map(i -> to - i + from - 1);
}
```

It correctly handles overflow as well, passing this test:

```java
@Test
public void testRevRange() {
    assertArrayEquals(revRange(0, 5).toArray(), new int[]{4, 3, 2, 1, 0});
    assertArrayEquals(revRange(-5, 0).toArray(), new int[]{-1, -2, -3, -4, -5});
    assertArrayEquals(revRange(1, 4).toArray(), new int[]{3, 2, 1});
    assertArrayEquals(revRange(0, 0).toArray(), new int[0]);
    assertArrayEquals(revRange(0, -1).toArray(), new int[0]);
    assertArrayEquals(revRange(MIN_VALUE, MIN_VALUE).toArray(), new int[0]);
    assertArrayEquals(revRange(MAX_VALUE, MAX_VALUE).toArray(), new int[0]);
    assertArrayEquals(revRange(MIN_VALUE, MIN_VALUE + 1).toArray(), new int[]{MIN_VALUE});
    assertArrayEquals(revRange(MAX_VALUE - 1, MAX_VALUE).toArray(), new int[]{MAX_VALUE - 1});
}
```

[Share](https://stackoverflow.com/a/34621495 "Short permalink to this answer")

Follow

[edited May 23, 2017 at 11:47](https://stackoverflow.com/posts/34621495/revisions "show all edits to this post")

[

![user avatar](/img/user/阅读库藏/assets/1658442695-bce878d0318dcbc306a18abf63ce5eee.png)

](https://stackoverflow.com/users/-1/community)

[Community](https://stackoverflow.com/users/-1/community)Bot

111 silver badge

answered Jan 5, 2016 at 21:25

[

![user avatar](/img/user/阅读库藏/assets/1658442695-52a711dd45d18cebd45e3e3078b0af56.png)

](https://stackoverflow.com/users/1237044/brandon)

[Brandon](https://stackoverflow.com/users/1237044/brandon)

2,2982525 silver badges3232 bronze badges

*   awesome and simple. Is this from some opensource utilty? (the Estreams thing) or some piece of your code?
    
    – [vach](https://stackoverflow.com/users/654799/vach "9,657 reputation")
    
    [Jan 6, 2016 at 9:47](#comment57005289_34621495)
    
*   Thanks! Unfortunately, I didn't intend to leak the `Estreams` name (I'm going to remove it from the post). It's one of our company's internal utility classes, which we use to supplement `java.util.stream.Stream`'s `static` methods.
    
    – [Brandon](https://stackoverflow.com/users/1237044/brandon "2,298 reputation")
    
    [Jan 6, 2016 at 20:54](#comment57031040_34621495)
    
*   4
    
    Okay…“awesome and simple”… Is there any case that this solution handles, which [Stuart Marks’ even simpler solution](http://stackoverflow.com/a/24011264/2711488) didn’t already handle more than one and a half year ago?
    
    – [Holger](https://stackoverflow.com/users/2711488/holger "269,089 reputation")
    
    [Jan 28, 2016 at 19:24](#comment57863442_34621495)
    
*   I just tested his solution with my test method above; it passes. I was unnecessarily avoiding overflow instead of embracing it as he did. I agree that his is better. I'll edit mine to reflect that.
    
    – [Brandon](https://stackoverflow.com/users/1237044/brandon "2,298 reputation")
    
    [Jan 28, 2016 at 23:43](#comment57871596_34621495)
    
*   1
    
    @vach, you may use by [`StreamEx`](https://github.com/amaembo/streamex) specifying step: `IntStreamEx.rangeClosed(from-1, to, -1)`
    
    – [Tagir Valeev](https://stackoverflow.com/users/4856258/tagir-valeev "93,333 reputation")
    
    [Jan 29, 2016 at 1:57](#comment57873933_34621495)
    

[Show **4** more comments](# "Expand to show all comments on this post")

22

[](https://stackoverflow.com/posts/27404228/timeline)

without external lib...

```java
import java.util.List;
import java.util.Collections;
import java.util.stream.Collector;

public class MyCollectors {

    public static <T> Collector<T, ?, List<T>> toListReversed() {
        return Collectors.collectingAndThen(Collectors.toList(), l -> {
            Collections.reverse(l);
            return l;
        });
    }

}
```

[Share](https://stackoverflow.com/a/27404228 "Short permalink to this answer")

Follow

answered Dec 10, 2014 at 15:02

[

![user avatar](/img/user/阅读库藏/assets/1658442695-9b4ecd59c5fb861c2f788b0e2e5e3089.png)

](https://stackoverflow.com/users/460175/comonad)

[comonad](https://stackoverflow.com/users/460175/comonad)

4,98822 gold badges3131 silver badges3030 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

18

[](https://stackoverflow.com/posts/63806719/timeline)

**How NOT to do it:**

*   Don't use `.sorted(Comparator.reverseOrder())` or `.sorted(Collections.reverseOrder())`, because it will just sort elements in the descending order.  
    Using it for given Integer input:  
    `[1, 4, 2, 5, 3]`  
    the output would be as follows:  
    `[5, 4, 3, 2, 1]`  
    For String input:  
    `["A", "D", "B", "E", "C"]`  
    the output would be as follows:  
    `[E, D, C, B, A]`
*   Don't use `.sorted((a, b) -> -1)` (explanation at the end)

**The easiest way to do it properly:**

```java
List<Integer> list = Arrays.asList(1, 4, 2, 5, 3);
Collections.reverse(list);
System.out.println(list);
```

Output:  
`[3, 5, 2, 4, 1]`

The same for `String`:

```java
List<String> stringList = Arrays.asList("A", "D", "B", "E", "C");
Collections.reverse(stringList);
System.out.println(stringList);
```

Output:  
`[C, E, B, D, A]`

**Don't use `.sorted((a, b) -> -1)`!**  
It breaks comparator contract and might work only for some cases ie. only on single thread but not in parallel.  
yankee explanation:

> `(a, b) -> -1` breaks the contract for `Comparator`. Whether this works depends on the implementation of the sort algorithm. The next release of the JVM might break this. Actually I can already break this reproduciblly on my machine using `IntStream.range(0, 10000).parallel().boxed().sorted((a, b) -> -1).forEachOrdered(System.out::println);`

```java
//Don't use this!!!
List<Integer> list = Arrays.asList(1, 4, 2, 5, 3);
List<Integer> reversedList = list.stream()
        .sorted((a, b) -> -1)
        .collect(Collectors.toList());
System.out.println(reversedList);
```

Output in positive case:  
`[3, 5, 2, 4, 1]`

Possible output in parallel stream or with other JVM implementation:  
`[4, 1, 2, 3, 5]`

The same for `String`:

```java
//Don't use this!!!
List<String> stringList = Arrays.asList("A", "D", "B", "E", "C");
List<String> reversedStringList = stringList.stream()
        .sorted((a, b) -> -1)
        .collect(Collectors.toList());
System.out.println(reversedStringList);
```

Output in positive case:  
`[C, E, B, D, A]`

Possible output in parallel stream or with other JVM implementation:  
`[A, E, B, D, C]`

[Share](https://stackoverflow.com/a/63806719 "Short permalink to this answer")

Follow

[edited Nov 30, 2020 at 10:51](https://stackoverflow.com/posts/63806719/revisions "show all edits to this post")

answered Sep 9, 2020 at 7:29

[

![user avatar](/img/user/阅读库藏/assets/1658442695-7fee837300bb32e238fda8a8e43f18d8.png)

](https://stackoverflow.com/users/1673775/luke)

[luke](https://stackoverflow.com/users/1673775/luke)

2,5252828 silver badges3939 bronze badges

*   2
    
    `(a, b) -> -1` breaks the contract for `Comparator`. Whether this works depends on the implementation of the sort algorithm. The next release of the JVM might break this. Actually I can already break this reproduciblly on my machine using `IntStream.range(0, 10000).parallel().boxed().sorted((a, b) -> -1).forEachOrdered(System.out::println);`
    
    – [yankee](https://stackoverflow.com/users/327301/yankee "36,614 reputation")
    
    [Nov 12, 2020 at 13:09](#comment114577416_63806719)
    
*   @yankee Yes you are right. It breaks `Comparator` contract and is kind of misuse of `Comparator`. It works fine for single thread, but even if you don't use `.parallel()` you can't rely on it, because you don't know how virtual machine will run this and you don't know which sorting algorithm will be used (maybe even sorting algorithm working on single thread might break with this implementation). Thank you for your comment. I will revise my answer in my free time.
    
    – [luke](https://stackoverflow.com/users/1673775/luke "2,525 reputation")
    
    [Nov 19, 2020 at 11:26](#comment114761293_63806719)
    
*   I revised my answer according to yankee comment. Now the answer should be correct.
    
    – [luke](https://stackoverflow.com/users/1673775/luke "2,525 reputation")
    
    [Nov 30, 2020 at 10:46](#comment115040967_63806719)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

12

[](https://stackoverflow.com/posts/24717422/timeline)

You could define your own collector that collects the elements in reverse order:

```java
public static <T> Collector<T, List<T>, List<T>> inReverse() {
    return Collector.of(
        ArrayList::new,
        (l, t) -> l.add(t),
        (l, r) -> {l.addAll(r); return l;},
        Lists::<T>reverse);
}
```

And use it like:

```java
stream.collect(inReverse()).forEach(t -> ...)
```

I use an ArrayList in forward order to efficiently insert collect the items (at the end of the list), and Guava Lists.reverse to efficiently give a reversed view of the list without making another copy of it.

Here are some test cases for the custom collector:

```java
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.*;

import java.util.ArrayList;
import java.util.List;
import java.util.function.BiConsumer;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.function.Supplier;
import java.util.stream.Collector;

import org.hamcrest.Matchers;
import org.junit.Test;

import com.google.common.collect.Lists;

public class TestReverseCollector {
    private final Object t1 = new Object();
    private final Object t2 = new Object();
    private final Object t3 = new Object();
    private final Object t4 = new Object();

    private final Collector<Object, List<Object>, List<Object>> inReverse = inReverse();
    private final Supplier<List<Object>> supplier = inReverse.supplier();
    private final BiConsumer<List<Object>, Object> accumulator = inReverse.accumulator();
    private final Function<List<Object>, List<Object>> finisher = inReverse.finisher();
    private final BinaryOperator<List<Object>> combiner = inReverse.combiner();

    @Test public void associative() {
        final List<Object> a1 = supplier.get();
        accumulator.accept(a1, t1);
        accumulator.accept(a1, t2);
        final List<Object> r1 = finisher.apply(a1);

        final List<Object> a2 = supplier.get();
        accumulator.accept(a2, t1);
        final List<Object> a3 = supplier.get();
        accumulator.accept(a3, t2);
        final List<Object> r2 = finisher.apply(combiner.apply(a2, a3));

        assertThat(r1, Matchers.equalTo(r2));
    }

    @Test public void identity() {
        final List<Object> a1 = supplier.get();
        accumulator.accept(a1, t1);
        accumulator.accept(a1, t2);
        final List<Object> r1 = finisher.apply(a1);

        final List<Object> a2 = supplier.get();
        accumulator.accept(a2, t1);
        accumulator.accept(a2, t2);
        final List<Object> r2 = finisher.apply(combiner.apply(a2, supplier.get()));

        assertThat(r1, equalTo(r2));
    }

    @Test public void reversing() throws Exception {
        final List<Object> a2 = supplier.get();
        accumulator.accept(a2, t1);
        accumulator.accept(a2, t2);

        final List<Object> a3 = supplier.get();
        accumulator.accept(a3, t3);
        accumulator.accept(a3, t4);

        final List<Object> r2 = finisher.apply(combiner.apply(a2, a3));

        assertThat(r2, contains(t4, t3, t2, t1));
    }

    public static <T> Collector<T, List<T>, List<T>> inReverse() {
        return Collector.of(
            ArrayList::new,
            (l, t) -> l.add(t),
            (l, r) -> {l.addAll(r); return l;},
            Lists::<T>reverse);
    }
}
```

[Share](https://stackoverflow.com/a/24717422 "Short permalink to this answer")

Follow

answered Jul 12, 2014 at 21:19

[

![user avatar](/img/user/阅读库藏/assets/1658442695-64822d1538071bcf3bea813761318428.png)

](https://stackoverflow.com/users/72810/lexicalscope)

[lexicalscope](https://stackoverflow.com/users/72810/lexicalscope)

6,79066 gold badges3636 silver badges5656 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

12

[](https://stackoverflow.com/posts/41911784/timeline)

If implemented `Comparable<T>` (ex. `Integer`, `String`, `Date`), you can do it using `Comparator.reverseOrder()`.

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4);
list.stream()
     .sorted(Comparator.reverseOrder())
     .forEach(System.out::println);
```

[Share](https://stackoverflow.com/a/41911784 "Short permalink to this answer")

Follow

[edited Feb 10, 2019 at 21:34](https://stackoverflow.com/posts/41911784/revisions "show all edits to this post")

[

![user avatar](/img/user/阅读库藏/assets/1658442695-541a105345d573de009c1d2cae250dde.jpg)

](https://stackoverflow.com/users/4298200/neuron)

[Neuron](https://stackoverflow.com/users/4298200/neuron)

4,46744 gold badges3232 silver badges5353 bronze badges

answered Jan 28, 2017 at 15:57

[

![user avatar](/img/user/阅读库藏/assets/1658442695-311fd21904f0efa655b734b800ef08e2.jpg)

](https://stackoverflow.com/users/1932017/yujisoftware)

[YujiSoftware](https://stackoverflow.com/users/1932017/yujisoftware)

1,1111212 silver badges1414 bronze badges

*   24
    
    This doesn't reverse the stream. It sorts the stream in reverse order. So if you had `Stream.of(1,3,2)` the result would be `Stream.of(3,2,1)` NOT `Stream.of(2,3,1)`
    
    – [wilmol](https://stackoverflow.com/users/6122976/wilmol "953 reputation")
    
    [Nov 15, 2019 at 3:08](#comment104008332_41911784)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

9

[](https://stackoverflow.com/posts/30786975/timeline)

[cyclops-react](https://github.com/aol/cyclops-react) StreamUtils has a reverse Stream method ([javadoc](http://static.javadoc.io/com.aol.simplereact/cyclops-react/1.0.0-RC1/com/aol/cyclops/util/stream/StreamUtils.html)).

```java
  StreamUtils.reverse(Stream.of("1", "2", "20", "3"))
             .forEach(System.out::println);
```

It works by collecting to an ArrayList and then making use of the ListIterator class which can iterate in either direction, to iterate backwards over the list.

If you already have a List, it will be more efficient

```java
  StreamUtils.reversedStream(Arrays.asList("1", "2", "20", "3"))
             .forEach(System.out::println);
```

[Share](https://stackoverflow.com/a/30786975 "Short permalink to this answer")

Follow

[edited Feb 29, 2016 at 11:15](https://stackoverflow.com/posts/30786975/revisions "show all edits to this post")

answered Jun 11, 2015 at 16:58

[

![user avatar](/img/user/阅读库藏/assets/1658442695-d2058a22c8c8112649413c3be50811ee.png)

](https://stackoverflow.com/users/4653367/john-mcclean)

[John McClean](https://stackoverflow.com/users/4653367/john-mcclean)

5,02711 gold badge2121 silver badges3030 bronze badges

*   1
    
    Nice didnt know about this project
    
    – [vach](https://stackoverflow.com/users/654799/vach "9,657 reputation")
    
    [Jun 11, 2015 at 20:06](#comment49630361_30786975)
    
*   1
    
    cyclops now also comes with [Spliterators](http://static.javadoc.io/com.aol.cyclops/cyclops-sequence-api/6.0.1/com/aol/cyclops/sequence/spliterators/package-frame.html) for efficient Stream reversal (currently for Ranges, arrays and Lists). Creating cyclops [SequenceM](http://static.javadoc.io/com.aol.cyclops/cyclops-sequence-api/6.0.1/com/aol/cyclops/sequence/SequenceM.html) Stream extension with SequenceM.of, SequenceM.range or SequenceM.fromList will automatically take advantage of efficiently reversible spliterators.
    
    – [John McClean](https://stackoverflow.com/users/4653367/john-mcclean "5,027 reputation")
    
    [Sep 15, 2015 at 21:10](#comment53044913_30786975)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

7

[](https://stackoverflow.com/posts/36208024/timeline)

I would suggest using [jOOλ](https://github.com/jOOQ/jOOL), it's a great library that adds lots of useful functionality to Java 8 streams and lambdas.

You can then do the following:

```javascript
List<Integer> list = Arrays.asList(1,2,3,4);    
Seq.seq(list).reverse().forEach(System.out::println)
```

Simple as that. It's a pretty lightweight library, and well worth adding to any Java 8 project.

[Share](https://stackoverflow.com/a/36208024 "Short permalink to this answer")

Follow

answered Mar 24, 2016 at 18:59

[

![user avatar](/img/user/阅读库藏/assets/1658442695-c2046a53a53c505eccf4b3b3c25c9c3c.jpg)

](https://stackoverflow.com/users/680021/lukens)

[lukens](https://stackoverflow.com/users/680021/lukens)

47033 silver badges99 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

6

[](https://stackoverflow.com/posts/24010541/timeline)

Here's the solution I've come up with:

```java
private static final Comparator<Integer> BY_ASCENDING_ORDER = Integer::compare;
private static final Comparator<Integer> BY_DESCENDING_ORDER = BY_ASCENDING_ORDER.reversed();
```

then using those comparators:

```java
IntStream.range(-range, 0).boxed().sorted(BY_DESCENDING_ORDER).forEach(// etc...
```

[Share](https://stackoverflow.com/a/24010541 "Short permalink to this answer")

Follow

answered Jun 3, 2014 at 8:33

[

![user avatar](/img/user/阅读库藏/assets/1658442695-82e6ad47632348bc7d1c4ac5e8def045.jpg)

](https://stackoverflow.com/users/654799/vach)

[vach](https://stackoverflow.com/users/654799/vach)

9,6571111 gold badges6060 silver badges9797 bronze badges

*   2
    
    This is only an answer to your "specific question" but not to your "general question".
    
    – [chiccodoro](https://stackoverflow.com/users/55787/chiccodoro "14,089 reputation")
    
    [Jun 3, 2014 at 8:55](#comment37007147_24010541)
    
*   13
    
    [`Collections.reverseOrder()`](http://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#reverseOrder--) exists since Java 1.2 and works with `Integer`…
    
    – [Holger](https://stackoverflow.com/users/2711488/holger "269,089 reputation")
    
    [Jun 3, 2014 at 11:36](#comment37013064_24010541)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

5

[](https://stackoverflow.com/posts/48752167/timeline)

How about this utility method?

```java
public static <T> Stream<T> getReverseStream(List<T> list) {
    final ListIterator<T> listIt = list.listIterator(list.size());
    final Iterator<T> reverseIterator = new Iterator<T>() {
        @Override
        public boolean hasNext() {
            return listIt.hasPrevious();
        }

        @Override
        public T next() {
            return listIt.previous();
        }
    };
    return StreamSupport.stream(Spliterators.spliteratorUnknownSize(
            reverseIterator,
            Spliterator.ORDERED | Spliterator.IMMUTABLE), false);
}
```

Seems to work with all cases without duplication.

[Share](https://stackoverflow.com/a/48752167 "Short permalink to this answer")

Follow

answered Feb 12, 2018 at 17:28

[

![user avatar](/img/user/阅读库藏/assets/1658442695-6509ab3843ce957434a9294efeeb4ff6.png)

](https://stackoverflow.com/users/1966569/jerome)

[Jerome](https://stackoverflow.com/users/1966569/jerome)

1,21522 gold badges1212 silver badges2323 bronze badges

*   1
    
    I like this solution a lot. Most answers split into two categories: (1) Reverse the collection and .stream() it, (2) Appeal to custom collectors. Both are absolutely unnecessary. Otherwise, it would've testified about some serious language expressiveness issue in JDK 8 itself. And your answer proves the opposite :)
    
    – [vitrums](https://stackoverflow.com/users/893369/vitrums "502 reputation")
    
    [Jul 6, 2020 at 18:44](#comment110988874_48752167)
    
*   (see [Adrian's May '19 answer](https://stackoverflow.com/a/56289986/3789665), too.)
    
    – [greybeard](https://stackoverflow.com/users/3789665/greybeard "2,102 reputation")
    
    [Jan 10 at 10:28](#comment124895089_48752167)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

4

[](https://stackoverflow.com/posts/49846647/timeline)

With regard to the specific question of generating a reverse `IntStream`:

starting from **Java 9** you can use the three-argument version of the [`IntStream.iterate(...)`](https://docs.oracle.com/javase/9/docs/api/java/util/stream/IntStream.html#iterate-int-java.util.function.IntPredicate-java.util.function.IntUnaryOperator-):

```java
IntStream.iterate(10, x -> x >= 0, x -> x - 1).forEach(System.out::println);

// Out: 10 9 8 7 6 5 4 3 2 1 0
```

where:

`IntStream.iterate​(int seed, IntPredicate hasNext, IntUnaryOperator next);`

*   `seed` - the initial element;
*   `hasNext` - a predicate to apply to elements to determine when the stream must terminate;
*   `next` - a function to be applied to the previous element to produce a new element.

[Share](https://stackoverflow.com/a/49846647 "Short permalink to this answer")

Follow

[edited Jun 2, 2019 at 22:49](https://stackoverflow.com/posts/49846647/revisions "show all edits to this post")

answered Apr 15, 2018 at 20:48

[

![user avatar](/img/user/阅读库藏/assets/1658442695-d666055cf5a14a3ee77c56b02bf6065f.jpg)

](https://stackoverflow.com/users/2668232/oleksandr-pyrohov)

[Oleksandr Pyrohov](https://stackoverflow.com/users/2668232/oleksandr-pyrohov)

14.1k55 gold badges5858 silver badges8787 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

3

[](https://stackoverflow.com/posts/46845327/timeline)

**Simplest way** (simple collect - supports parallel streams):

```java
public static <T> Stream<T> reverse(Stream<T> stream) {
    return stream
            .collect(Collector.of(
                    () -> new ArrayDeque<T>(),
                    ArrayDeque::addFirst,
                    (q1, q2) -> { q2.addAll(q1); return q2; })
            )
            .stream();
}
```

**Advanced way** (supports parallel streams in an ongoing way):

```java
public static <T> Stream<T> reverse(Stream<T> stream) {
    Objects.requireNonNull(stream, "stream");

    class ReverseSpliterator implements Spliterator<T> {
        private Spliterator<T> spliterator;
        private final Deque<T> deque = new ArrayDeque<>();

        private ReverseSpliterator(Spliterator<T> spliterator) {
            this.spliterator = spliterator;
        }

        @Override
        @SuppressWarnings({"StatementWithEmptyBody"})
        public boolean tryAdvance(Consumer<? super T> action) {
            while(spliterator.tryAdvance(deque::addFirst));
            if(!deque.isEmpty()) {
                action.accept(deque.remove());
                return true;
            }
            return false;
        }

        @Override
        public Spliterator<T> trySplit() {
            // After traveling started the spliterator don't contain elements!
            Spliterator<T> prev = spliterator.trySplit();
            if(prev == null) {
                return null;
            }

            Spliterator<T> me = spliterator;
            spliterator = prev;
            return new ReverseSpliterator(me);
        }

        @Override
        public long estimateSize() {
            return spliterator.estimateSize();
        }

        @Override
        public int characteristics() {
            return spliterator.characteristics();
        }

        @Override
        public Comparator<? super T> getComparator() {
            Comparator<? super T> comparator = spliterator.getComparator();
            return (comparator != null) ? comparator.reversed() : null;
        }

        @Override
        public void forEachRemaining(Consumer<? super T> action) {
            // Ensure that tryAdvance is called at least once
            if(!deque.isEmpty() || tryAdvance(action)) {
                deque.forEach(action);
            }
        }
    }

    return StreamSupport.stream(new ReverseSpliterator(stream.spliterator()), stream.isParallel());
}
```

Note you can quickly extends to other type of streams (IntStream, ...).

**Testing:**

```java
// Use parallel if you wish only
revert(Stream.of("One", "Two", "Three", "Four", "Five", "Six").parallel())
    .forEachOrdered(System.out::println);
```

**Results:**

```java
Six
Five
Four
Three
Two
One
```

**Additional notes:** The `simplest way` it isn't so useful when used with other stream operations (the collect join breaks the parallelism). The `advance way` doesn't have that issue, and it keeps also the initial characteristics of the stream, for example `SORTED`, and so, it's the way to go to use with other stream operations after the reverse.

[Share](https://stackoverflow.com/a/46845327 "Short permalink to this answer")

Follow

[edited Oct 20, 2017 at 11:47](https://stackoverflow.com/posts/46845327/revisions "show all edits to this post")

answered Oct 20, 2017 at 8:39

[

![user avatar](/img/user/阅读库藏/assets/1658442695-ff075f1beb08cb5aec90d798e7dc8bb3.jpg)

](https://stackoverflow.com/users/8718583/tet)

[Tet](https://stackoverflow.com/users/8718583/tet)

1,13599 silver badges1414 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

3

[](https://stackoverflow.com/posts/59019747/timeline)

```java
List newStream = list.stream().sorted(Collections.reverseOrder()).collect(Collectors.toList());
        newStream.forEach(System.out::println);
```

[Share](https://stackoverflow.com/a/59019747 "Short permalink to this answer")

Follow

[edited Feb 17, 2020 at 7:16](https://stackoverflow.com/posts/59019747/revisions "show all edits to this post")

[

![user avatar](/img/user/阅读库藏/assets/1658442695-0fe6af32457b8a64990cc181005d0b99.jpg)

](https://stackoverflow.com/users/3992939/c0der)

[c0der](https://stackoverflow.com/users/3992939/c0der)

18.1k66 gold badges3131 silver badges6262 bronze badges

answered Nov 24, 2019 at 16:18

[

![user avatar](/img/user/阅读库藏/assets/1658442695-9fb09c8829d5143e257be9f7d48b5962.jpg)

](https://stackoverflow.com/users/1651744/siddmuk2005)

[siddmuk2005](https://stackoverflow.com/users/1651744/siddmuk2005)

17911 silver badge99 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

2

[](https://stackoverflow.com/posts/32299976/timeline)

One could write a collector that collects elements in reversed order:

```java
public static <T> Collector<T, ?, Stream<T>> reversed() {
    return Collectors.collectingAndThen(Collectors.toList(), list -> {
        Collections.reverse(list);
        return list.stream();
    });
}
```

And use it like this:

```java
Stream.of(1, 2, 3, 4, 5).collect(reversed()).forEach(System.out::println);
```

**Original answer** (contains a bug - it does not work correctly for parallel streams):

A general purpose stream reverse method could look like:

```java
public static <T> Stream<T> reverse(Stream<T> stream) {
    LinkedList<T> stack = new LinkedList<>();
    stream.forEach(stack::push);
    return stack.stream();
}
```

[Share](https://stackoverflow.com/a/32299976 "Short permalink to this answer")

Follow

[edited Jul 26, 2016 at 4:33](https://stackoverflow.com/posts/32299976/revisions "show all edits to this post")

answered Aug 30, 2015 at 18:49

[

![user avatar](/img/user/阅读库藏/assets/1658442695-2470ec144393a792c39f131c259b3e0f.png)

](https://stackoverflow.com/users/1707382/slavast)

[SlavaSt](https://stackoverflow.com/users/1707382/slavast)

1,43311 gold badge1515 silver badges1919 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

2

[](https://stackoverflow.com/posts/45818350/timeline)

For reference I was looking at the same problem, I wanted to join the string value of stream elements in the reverse order.

itemList = { last, middle, first } => first,middle,last

I started to use an intermediate collection with `collectingAndThen` from _comonad_ or the `ArrayDeque` collector of _Stuart Marks_, although I wasn't happy with intermediate collection, and streaming again

```java
itemList.stream()
        .map(TheObject::toString)
        .collect(Collectors.collectingAndThen(Collectors.toList(),
                                              strings -> {
                                                      Collections.reverse(strings);
                                                      return strings;
                                              }))
        .stream()
        .collect(Collector.joining());
```

So I iterated over Stuart Marks answer that was using the `Collector.of` factory, that has the interesting _finisher_ lambda.

```java
itemList.stream()
        .collect(Collector.of(StringBuilder::new,
                             (sb, o) -> sb.insert(0, o),
                             (r1, r2) -> { r1.insert(0, r2); return r1; },
                             StringBuilder::toString));
```

Since in this case the stream is not parallel, the combiner is not relevant that much, I'm using `insert` anyway for the sake of code consistency but it does not matter as it would depend of which stringbuilder is built first.

I looked at the StringJoiner, however it does not have an `insert` method.

[Share](https://stackoverflow.com/a/45818350 "Short permalink to this answer")

Follow

answered Aug 22, 2017 at 12:58

[

![user avatar](/img/user/阅读库藏/assets/1658442695-a33830feb7b348f9499f5ca92f9aa62f.jpg)

](https://stackoverflow.com/users/48136/brice)

[Brice](https://stackoverflow.com/users/48136/brice)

37.4k99 gold badges8383 silver badges102102 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

2

[](https://stackoverflow.com/posts/50376064/timeline)

Not purely Java8 but if you use guava's Lists.reverse() method in conjunction, you can easily achieve this:

```java
List<Integer> list = Arrays.asList(1,2,3,4);
Lists.reverse(list).stream().forEach(System.out::println);
```

[Share](https://stackoverflow.com/a/50376064 "Short permalink to this answer")

Follow

answered May 16, 2018 at 16:41

[

![user avatar](/img/user/阅读库藏/assets/1658442695-7c05438dd4e06a407c121c135f4262b3.png)

](https://stackoverflow.com/users/107216/jeffrey)

[Jeffrey](https://stackoverflow.com/users/107216/jeffrey)

97822 gold badges1212 silver badges2424 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

2

[](https://stackoverflow.com/posts/53866886/timeline)

`ArrayDeque` are faster in the stack than a Stack or LinkedList. "push()" inserts elements at the front of the Deque

```java
 protected <T> Stream<T> reverse(Stream<T> stream) {
    ArrayDeque<T> stack = new ArrayDeque<>();
    stream.forEach(stack::push);
    return stack.stream();
}
```

[Share](https://stackoverflow.com/a/53866886 "Short permalink to this answer")

Follow

[edited Dec 20, 2018 at 11:32](https://stackoverflow.com/posts/53866886/revisions "show all edits to this post")

[

![user avatar](/img/user/阅读库藏/assets/1658442695-91b40149a6065ac54c5b5af1a0804bae.jpg)

](https://stackoverflow.com/users/8338128/prakash-pazhanisamy)

[Prakash Pazhanisamy](https://stackoverflow.com/users/8338128/prakash-pazhanisamy)

98111 gold badge1414 silver badges2525 bronze badges

answered Dec 20, 2018 at 10:32

[

![user avatar](/img/user/阅读库藏/assets/1658442695-a6f6701d0c2c91d57adf55be70a53213.jpg)

](https://stackoverflow.com/users/4278474/dotista)

[Dotista](https://stackoverflow.com/users/4278474/dotista)

33722 silver badges99 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

2

[](https://stackoverflow.com/posts/55932731/timeline)

Reversing string or any Array

```java
(Stream.of("abcdefghijklm 1234567".split("")).collect(Collectors.collectingAndThen(Collectors.toList(),list -> {Collections.reverse(list);return list;}))).stream().forEach(System.out::println);
```

split can be modified based on the delimiter or space

[Share](https://stackoverflow.com/a/55932731 "Short permalink to this answer")

Follow

[edited May 1, 2019 at 8:28](https://stackoverflow.com/posts/55932731/revisions "show all edits to this post")

[

![user avatar](/img/user/阅读库藏/assets/1658442695-0d9d85bafe8f49593713ae6dd89c5466.png)

](https://stackoverflow.com/users/1600162/hessam)

[Hessam](https://stackoverflow.com/users/1600162/hessam)

1,25911 gold badge2525 silver badges4141 bronze badges

answered May 1, 2019 at 7:07

[

![user avatar](/img/user/阅读库藏/assets/1658442695-964e70c96fead9f234c1b2d8ced559bb.jpg)

](https://stackoverflow.com/users/11435928/sai-manoj)

[sai manoj](https://stackoverflow.com/users/11435928/sai-manoj)

2111 bronze badge

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

2

[](https://stackoverflow.com/posts/65651454/timeline)

```java
How about reversing the Collection backing the stream prior?

import java.util.Collections;
import java.util.List;

public void reverseTest(List<Integer> sampleCollection) {
    Collections.reverse(sampleCollection); // remember this reverses the elements in the list, so if you want the original input collection to remain untouched clone it first.

    sampleCollection.stream().forEach(item -> {
      // you op here
    });
}
```

[Share](https://stackoverflow.com/a/65651454 "Short permalink to this answer")

Follow

answered Jan 10, 2021 at 8:41

[

![user avatar](/img/user/阅读库藏/assets/1658442695-2ad0e904549a096e7d30258b93df1c6e.jpg)

](https://stackoverflow.com/users/9286715/levenshtein)

[levenshtein](https://stackoverflow.com/users/9286715/levenshtein)

68177 silver badges44 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

1

[](https://stackoverflow.com/posts/46352674/timeline)

Answering specific question of reversing with IntStream, below worked for me:

```java
IntStream.range(0, 10)
  .map(x -> x * -1)
  .sorted()
  .map(Math::abs)
  .forEach(System.out::println);
```

[Share](https://stackoverflow.com/a/46352674 "Short permalink to this answer")

Follow

answered Sep 21, 2017 at 20:29

[

![user avatar](/img/user/阅读库藏/assets/1658442695-6192843296abd4c4468586cdf241f9f3.png)

](https://stackoverflow.com/users/428532/girish-jain)

[Girish Jain](https://stackoverflow.com/users/428532/girish-jain)

55544 silver badges99 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

1

[](https://stackoverflow.com/posts/56289986/timeline)

the simplest solution is using `List::listIterator` and `Stream::generate`

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
ListIterator<Integer> listIterator = list.listIterator(list.size());

Stream.generate(listIterator::previous)
      .limit(list.size())
      .forEach(System.out::println);
```

[Share](https://stackoverflow.com/a/56289986 "Short permalink to this answer")

Follow

answered May 24, 2019 at 9:44

[

![user avatar](/img/user/阅读库藏/assets/1658442695-cd6c9e8251cc3793d086d3b7f8205fd5.png)

](https://stackoverflow.com/users/8198056/adrian)

[Adrian](https://stackoverflow.com/users/8198056/adrian)

2,7441212 silver badges2525 bronze badges

*   It's worth adding that `Stream.generate()` generates in infinite stream, so the call to `limit()` is very important here.
    
    – [andrebrait](https://stackoverflow.com/users/2921256/andrebrait "447 reputation")
    
    [Jul 5, 2019 at 8:43](#comment100344397_56289986)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

1

[](https://stackoverflow.com/posts/60621923/timeline)

This method works with any Stream and is Java 8 compliant:

```java
Stream<Integer> myStream = Stream.of(1, 2, 3, 4, 5);
myStream.reduce(Stream.empty(),
        (Stream<Integer> a, Integer b) -> Stream.concat(Stream.of(b), a),
        (a, b) -> Stream.concat(b, a))
        .forEach(System.out::println);
```

[Share](https://stackoverflow.com/a/60621923 "Short permalink to this answer")

Follow

[edited Mar 18, 2020 at 15:42](https://stackoverflow.com/posts/60621923/revisions "show all edits to this post")

answered Mar 10, 2020 at 16:26

[

![user avatar](/img/user/阅读库藏/assets/1658442695-1a013fca8a02adc4e7fa5d163fe25ba7.jpg)

](https://stackoverflow.com/users/177832/david-larochette)

[David Larochette](https://stackoverflow.com/users/177832/david-larochette)

1,1611010 silver badges1818 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/48136802/timeline)

This is how I do it.

I don't like the idea of creating a new collection and reverse iterating it.

The IntStream#map idea is pretty neat, but I prefer the IntStream#iterate method, for I think the idea of a countdown to Zero better expressed with the iterate method and easier to understand in terms of walking the array from back to front.

```java
import static java.lang.Math.max;

private static final double EXACT_MATCH = 0d;

public static IntStream reverseStream(final int[] array) {
    return countdownFrom(array.length - 1).map(index -> array[index]);
}

public static DoubleStream reverseStream(final double[] array) {
    return countdownFrom(array.length - 1).mapToDouble(index -> array[index]);
}

public static <T> Stream<T> reverseStream(final T[] array) {
    return countdownFrom(array.length - 1).mapToObj(index -> array[index]);
}

public static IntStream countdownFrom(final int top) {
    return IntStream.iterate(top, t -> t - 1).limit(max(0, (long) top + 1));
}
```

Here are some tests to prove it works:

```java
import static java.lang.Integer.MAX_VALUE;
import static org.junit.Assert.*;

@Test
public void testReverseStream_emptyArrayCreatesEmptyStream() {
    Assert.assertEquals(0, reverseStream(new double[0]).count());
}

@Test
public void testReverseStream_singleElementCreatesSingleElementStream() {
    Assert.assertEquals(1, reverseStream(new double[1]).count());
    final double[] singleElementArray = new double[] { 123.4 };
    assertArrayEquals(singleElementArray, reverseStream(singleElementArray).toArray(), EXACT_MATCH);
}

@Test
public void testReverseStream_multipleElementsAreStreamedInReversedOrder() {
    final double[] arr = new double[] { 1d, 2d, 3d };
    final double[] revArr = new double[] { 3d, 2d, 1d };
    Assert.assertEquals(arr.length, reverseStream(arr).count());
    Assert.assertArrayEquals(revArr, reverseStream(arr).toArray(), EXACT_MATCH);
}

@Test
public void testCountdownFrom_returnsAllElementsFromTopToZeroInReverseOrder() {
    assertArrayEquals(new int[] { 4, 3, 2, 1, 0 }, countdownFrom(4).toArray());
}

@Test
public void testCountdownFrom_countingDownStartingWithZeroOutputsTheNumberZero() {
    assertArrayEquals(new int[] { 0 }, countdownFrom(0).toArray());
}

@Test
public void testCountdownFrom_doesNotChokeOnIntegerMaxValue() {
    assertEquals(true, countdownFrom(MAX_VALUE).anyMatch(x -> x == MAX_VALUE));
}

@Test
public void testCountdownFrom_givesZeroLengthCountForNegativeValues() {
    assertArrayEquals(new int[0], countdownFrom(-1).toArray());
    assertArrayEquals(new int[0], countdownFrom(-4).toArray());
}
```

[Share](https://stackoverflow.com/a/48136802 "Short permalink to this answer")

Follow

answered Jan 7, 2018 at 11:47

[

![user avatar](/img/user/阅读库藏/assets/1658442695-cb37f09861c357be6c329293592d8aa5.jpg)

](https://stackoverflow.com/users/6198746/brixomatic)

[Brixomatic](https://stackoverflow.com/users/6198746/brixomatic)

34144 silver badges1313 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/53545594/timeline)

In all this I don't see the answer I would go to first.

This isn't exactly a direct answer to the question, but it's a potential solution to the problem.

Just build the list backwards in the first place. If you can, use a LinkedList instead of an ArrayList and when you add items use "Push" instead of add. The list will be built in the reverse order and will then stream correctly without any manipulation.

This won't fit cases where you are dealing with primitive arrays or lists that are already used in various ways but does work well in a surprising number of cases.

[Share](https://stackoverflow.com/a/53545594 "Short permalink to this answer")

Follow

answered Nov 29, 2018 at 18:40

[

![user avatar](/img/user/阅读库藏/assets/1658442695-125f0b3008fc32d4a21989f60e6d95de.png)

](https://stackoverflow.com/users/12943/bill-k)

[Bill K](https://stackoverflow.com/users/12943/bill-k)

61.3k1717 gold badges101101 silver badges153153 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/64421976/timeline)

Based on [@stuart-marks's answer](https://stackoverflow.com/a/24011264/6346531), but without casting, function returning stream of list elements starting from end:

```java
public static <T> Stream<T> reversedStream(List<T> tList) {
    final int size = tList.size();
    return IntStream.range(0, size)
            .mapToObj(i -> tList.get(size - 1 - i));
}

// usage
reversedStream(list).forEach(System.out::println);
```

[Share](https://stackoverflow.com/a/64421976 "Short permalink to this answer")

Follow

answered Oct 19, 2020 at 6:08

[

![user avatar](/img/user/阅读库藏/assets/1658442695-ce01ad856cbee6fc0d3c82efb814b344.png)

](https://stackoverflow.com/users/6346531/aksh1618)

[aksh1618](https://stackoverflow.com/users/6346531/aksh1618)

1,8331414 silver badges3434 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/70650767/timeline)

`What's the proper generic way to reverse a stream?`

If the stream does not specify an [_encounter order_](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/Spliterator.html#ORDERED), don't. (`!s.spliterator().hasCharacteristics(java.util.Spliterator.ORDERED)`)

[Share](https://stackoverflow.com/a/70650767 "Short permalink to this answer")

Follow

answered Jan 10 at 10:15

[

![user avatar](/img/user/阅读库藏/assets/1658442695-f5272176dc2a25d8d2ca288443c4531d.png)

](https://stackoverflow.com/users/3789665/greybeard)

[greybeard](https://stackoverflow.com/users/3789665/greybeard)

2,10277 gold badges2525 silver badges5959 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

\-1

[](https://stackoverflow.com/posts/33249912/timeline)

The most generic and the easiest way to reverse a list will be :

```java
public static <T> void reverseHelper(List<T> li){

 li.stream()
.sorted((x,y)-> -1)
.collect(Collectors.toList())
.forEach(System.out::println);

    }
```

[Share](https://stackoverflow.com/a/33249912 "Short permalink to this answer")

Follow

[edited Oct 21, 2015 at 2:46](https://stackoverflow.com/posts/33249912/revisions "show all edits to this post")

answered Oct 21, 2015 at 2:24

[

![user avatar](/img/user/阅读库藏/assets/1658442695-005d47d2491514e0e04c679ba6efd44f.jpg)

](https://stackoverflow.com/users/3188270/parmeshwor11)

[parmeshwor11](https://stackoverflow.com/users/3188270/parmeshwor11)

7177 bronze badges

*   4
    
    You violate the contract of `Comparator`. As a result nobody can guarantee you that this "trick" will work in any future Java version with any sorting algorithm. The same trick does not work for parallel stream, for example, as parallel sorting algorithm uses `Comparator` in different way. For sequential sort it works purely by chance. I would not recommend anybody to use this solution.
    
    – [Tagir Valeev](https://stackoverflow.com/users/4856258/tagir-valeev "93,333 reputation")
    
    [Oct 21, 2015 at 8:51](#comment54311098_33249912)
    
*   1
    
    Also it doesn't work when you set `System.setProperty("java.util.Arrays.useLegacyMergeSort", "true");`
    
    – [Tagir Valeev](https://stackoverflow.com/users/4856258/tagir-valeev "93,333 reputation")
    
    [Oct 21, 2015 at 8:53](#comment54311185_33249912)
    
*   I thought it was about just printing things in reverse order not in sorted order (ie. descending order), and it works with parallel stream too : `public static <T> void reverseHelper(List<T> li){ li.parallelStream() .sorted((x,y)->-1) .collect(Collectors.toList()) .forEach(System.out::println); }`
    
    – [parmeshwor11](https://stackoverflow.com/users/3188270/parmeshwor11 "71 reputation")
    
    [Oct 21, 2015 at 9:27](#comment54312493_33249912)
    
*   1
    
    Try `reverseHelper(IntStream.range(0, 8193).boxed().collect(Collectors.toList()))` (result may depend on number of cores though).
    
    – [Tagir Valeev](https://stackoverflow.com/users/4856258/tagir-valeev "93,333 reputation")
    
    [Oct 21, 2015 at 10:51](#comment54315755_33249912)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

\-1

[](https://stackoverflow.com/posts/35089581/timeline)

Java 8 way to do this:

```java
    List<Integer> list = Arrays.asList(1,2,3,4);
    Comparator<Integer> comparator = Integer::compare;
    list.stream().sorted(comparator.reversed()).forEach(System.out::println);
```

[Share](https://stackoverflow.com/a/35089581 "Short permalink to this answer")

Follow

answered Jan 29, 2016 at 16:38

[

![user avatar](/img/user/阅读库藏/assets/1658442695-8b484c8a7428671f527d739364b2eec2.jpg)

](https://stackoverflow.com/users/203204/antonio)

[Antonio](https://stackoverflow.com/users/203204/antonio)

11.7k1111 gold badges6060 silver badges8888 bronze badges

*   4
    
    This is sorting in reverse order, not reverting a list.
    
    – [Jochen Bedersdorfer](https://stackoverflow.com/users/560902/jochen-bedersdorfer "4,033 reputation")
    
    [May 23, 2016 at 20:42](#comment62308985_35089581)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

**[Highly active question](https://stackoverflow.com/help/privileges/protect-questions)**. Earn 10 reputation (not counting the [association bonus](https://meta.stackexchange.com/questions/141648/what-is-the-association-bonus-and-how-does-it-work)) in order to answer this question. The reputation requirement helps protect this question from spam and non-answer activity.

## Not the answer you're looking for? Browse other questions tagged [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [list](https://stackoverflow.com/questions/tagged/list "show questions tagged 'list'") [sorting](https://stackoverflow.com/questions/tagged/sorting "show questions tagged 'sorting'") [java-8](https://stackoverflow.com/questions/tagged/java-8 "show questions tagged 'java-8'") [java-stream](https://stackoverflow.com/questions/tagged/java-stream "show questions tagged 'java-stream'") or [ask your own question](https://stackoverflow.com/questions/ask).

The Overflow Blog

*   [How APIs can take the pain out of legacy system headaches (Ep. 465)](https://stackoverflow.blog/2022/07/20/opentext-api-legacy-systems-podcast-ep-465/?cb=1)
    
*   [Design patterns for asynchronous API communication](https://stackoverflow.blog/2022/07/21/event-driven-topic-design-using-kafka/?cb=1)
    

Featured on Meta

*   [Announcing the Stacks Editor Beta release!](https://meta.stackexchange.com/questions/380295/announcing-the-stacks-editor-beta-release?cb=1)
    
*   [Trending: A new answer sorting option](https://meta.stackoverflow.com/questions/418767/trending-a-new-answer-sorting-option?cb=1)
    
*   [The \[options\] tag is being burninated](https://meta.stackoverflow.com/questions/417320/the-options-tag-is-being-burninated?cb=1)
    

#### Linked

[

6

](https://stackoverflow.com/q/70632163?lq=1 "Question score (upvotes - downvotes)")[Stream API, is there way to iterate descending using instream.range?](https://stackoverflow.com/questions/70632163/stream-api-is-there-way-to-iterate-descending-using-instream-range?noredirect=1&lq=1)

[

2

](https://stackoverflow.com/q/53016601?lq=1 "Question score (upvotes - downvotes)")[How to use back range in stream API?](https://stackoverflow.com/questions/53016601/how-to-use-back-range-in-stream-api?noredirect=1&lq=1)

[

1

](https://stackoverflow.com/q/53028500?lq=1 "Question score (upvotes - downvotes)")[is there any kind of reverse\_iterator in List, just to reverse the List? (Java8 stream solution preferred)](https://stackoverflow.com/questions/53028500/is-there-any-kind-of-reverse-iterator-in-list-just-to-reverse-the-list-java8?noredirect=1&lq=1)

[

0

](https://stackoverflow.com/q/46968953?lq=1 "Question score (upvotes - downvotes)")[Java 8: Reverse an int array in one line (using Streams API)](https://stackoverflow.com/questions/46968953/java-8-reverse-an-int-array-in-one-line-using-streams-api?noredirect=1&lq=1)

[

0

](https://stackoverflow.com/q/70873415?lq=1 "Question score (upvotes - downvotes)")[Java8 Stream How to iterate through elements from back to front？](https://stackoverflow.com/questions/70873415/java8-stream-how-to-iterate-through-elements-from-back-to-front?noredirect=1&lq=1)

[

0

](https://stackoverflow.com/q/49862979?lq=1 "Question score (upvotes - downvotes)")[How can I reorder a list in LAMBDA?](https://stackoverflow.com/questions/49862979/how-can-i-reorder-a-list-in-lambda?noredirect=1&lq=1)

[

237

](https://stackoverflow.com/q/12949690?lq=1 "Question score (upvotes - downvotes)")[Java ArrayList how to add elements at the beginning](https://stackoverflow.com/questions/12949690/java-arraylist-how-to-add-elements-at-the-beginning?noredirect=1&lq=1)

[

228

](https://stackoverflow.com/q/6163166?lq=1 "Question score (upvotes - downvotes)")[Why is ArrayDeque better than LinkedList](https://stackoverflow.com/questions/6163166/why-is-arraydeque-better-than-linkedlist?noredirect=1&lq=1)

[

146

](https://stackoverflow.com/q/21426843?lq=1 "Question score (upvotes - downvotes)")[Get last element of Stream/List in a one-liner](https://stackoverflow.com/questions/21426843/get-last-element-of-stream-list-in-a-one-liner?noredirect=1&lq=1)

[

12

](https://stackoverflow.com/q/54031994?lq=1 "Question score (upvotes - downvotes)")[Reversing a Queue<Integer> and converting it into an int array](https://stackoverflow.com/questions/54031994/reversing-a-queueinteger-and-converting-it-into-an-int-array?noredirect=1&lq=1)

[See more linked questions](https://stackoverflow.com/questions/linked/24010109?lq=1)

#### Related

[

4173

](https://stackoverflow.com/q/40471?rq=1 "Question score (upvotes - downvotes)")[What are the differences between a HashMap and a Hashtable in Java?](https://stackoverflow.com/questions/40471/what-are-the-differences-between-a-hashmap-and-a-hashtable-in-java?rq=1)

[

7465

](https://stackoverflow.com/q/40480?rq=1 "Question score (upvotes - downvotes)")[Is Java "pass-by-reference" or "pass-by-value"?](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value?rq=1)

[

3795

](https://stackoverflow.com/q/46898?rq=1 "Question score (upvotes - downvotes)")[How do I efficiently iterate over each entry in a Java Map?](https://stackoverflow.com/questions/46898/how-do-i-efficiently-iterate-over-each-entry-in-a-java-map?rq=1)

[

4305

](https://stackoverflow.com/q/271526?rq=1 "Question score (upvotes - downvotes)")[Avoiding NullPointerException in Java](https://stackoverflow.com/questions/271526/avoiding-nullpointerexception-in-java?rq=1)

[

4541

](https://stackoverflow.com/q/309424?rq=1 "Question score (upvotes - downvotes)")[How do I read / convert an InputStream into a String in Java?](https://stackoverflow.com/questions/309424/how-do-i-read-convert-an-inputstream-into-a-string-in-java?rq=1)

[

3475

](https://stackoverflow.com/q/322715?rq=1 "Question score (upvotes - downvotes)")[When to use LinkedList over ArrayList in Java?](https://stackoverflow.com/questions/322715/when-to-use-linkedlist-over-arraylist-in-java?rq=1)

[

3937

](https://stackoverflow.com/q/363681?rq=1 "Question score (upvotes - downvotes)")[How do I generate random integers within a specific range in Java?](https://stackoverflow.com/questions/363681/how-do-i-generate-random-integers-within-a-specific-range-in-java?rq=1)

[

3381

](https://stackoverflow.com/q/5585779?rq=1 "Question score (upvotes - downvotes)")[How do I convert a String to an int in Java?](https://stackoverflow.com/questions/5585779/how-do-i-convert-a-string-to-an-int-in-java?rq=1)

[

3603

](https://stackoverflow.com/q/6470651?rq=1 "Question score (upvotes - downvotes)")[How can I create a memory leak in Java?](https://stackoverflow.com/questions/6470651/how-can-i-create-a-memory-leak-in-java?rq=1)

[

973

](https://stackoverflow.com/q/23079003?rq=1 "Question score (upvotes - downvotes)")[How to convert a Java 8 Stream to an Array?](https://stackoverflow.com/questions/23079003/how-to-convert-a-java-8-stream-to-an-array?rq=1)

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [How should I deal with coworkers not respecting my blocking off time in my calendar for work?](https://workplace.stackexchange.com/questions/186306/how-should-i-deal-with-coworkers-not-respecting-my-blocking-off-time-in-my-calen)
*   [Is a neuron's information processing more complex than a perceptron?](https://psychology.stackexchange.com/questions/28672/is-a-neurons-information-processing-more-complex-than-a-perceptron)
*   [Is there a suffix that means "like", or "resembling"?](https://latin.stackexchange.com/questions/18540/is-there-a-suffix-that-means-like-or-resembling)
*   [Movie about robotic child seeking to wake his mother](https://scifi.stackexchange.com/questions/265717/movie-about-robotic-child-seeking-to-wake-his-mother)
*   [Make 2 0 2 2 2 0 2 2 = 2022](https://puzzling.stackexchange.com/questions/117161/make-2-0-2-2-2-0-2-2-2022)
*   [Can't reverse the legend of tm\_bubbles](https://gis.stackexchange.com/questions/436529/cant-reverse-the-legend-of-tm-bubbles)
*   [What drives the appeal and nostalgia of Margaret Thatcher within UK Conservative Party?](https://politics.stackexchange.com/questions/74345/what-drives-the-appeal-and-nostalgia-of-margaret-thatcher-within-uk-conservative)
*   [Why don’t second unit directors tend to become full-fledged directors?](https://movies.stackexchange.com/questions/118106/why-don-t-second-unit-directors-tend-to-become-full-fledged-directors)
*   [Is the fact that ZFC implies that 1+1=2 an absolute truth?](https://philosophy.stackexchange.com/questions/92335/is-the-fact-that-zfc-implies-that-11-2-an-absolute-truth)
*   [What is this tool used for?](https://diy.stackexchange.com/questions/253256/what-is-this-tool-used-for)
*   [Data Imbalance: what would be an ideal number(ratio) of newly added class's data?](https://stats.stackexchange.com/questions/582686/data-imbalance-what-would-be-an-ideal-numberratio-of-newly-added-classs-data)
*   [How can recreate this bubble wrap effect on my photos?](https://graphicdesign.stackexchange.com/questions/157935/how-can-recreate-this-bubble-wrap-effect-on-my-photos)
*   [Is there a PRNG that visits every number exactly once, in a non-trivial bitspace, without repetition, without large memory usage, before it cycles?](https://cs.stackexchange.com/questions/153104/is-there-a-prng-that-visits-every-number-exactly-once-in-a-non-trivial-bitspace)
*   [Estimation of the attenuation of two waves on a linear sensor array](https://dsp.stackexchange.com/questions/83914/estimation-of-the-attenuation-of-two-waves-on-a-linear-sensor-array)
*   [Are shrivelled chilis safe to eat and process into chili flakes?](https://cooking.stackexchange.com/questions/121095/are-shrivelled-chilis-safe-to-eat-and-process-into-chili-flakes)
*   [Solving hyperbolic equation with parallelization in python by elucidating Mathematica algorithm](https://mathematica.stackexchange.com/questions/271087/solving-hyperbolic-equation-with-parallelization-in-python-by-elucidating-mathem)
*   [Is there a political faction in Russia publicly advocating for an immediate ceasefire?](https://politics.stackexchange.com/questions/74348/is-there-a-political-faction-in-russia-publicly-advocating-for-an-immediate-ceas)
*   [Do weekend days count as part of a vacation?](https://law.stackexchange.com/questions/82320/do-weekend-days-count-as-part-of-a-vacation)
*   [Crushed Aluminum Dryer Vent](https://diy.stackexchange.com/questions/253279/crushed-aluminum-dryer-vent)
*   [How to explain mathematically 2.4 GHz and 5 GHz WiFi coverage and maximum range?](https://electronics.stackexchange.com/questions/628264/how-to-explain-mathematically-2-4-ghz-and-5-ghz-wifi-coverage-and-maximum-range)
*   [Story: man purchases plantation on planet, finds 'unstoppable' infestation, uses science, electrolyses water for oxygen, 1970s-1980s](https://scifi.stackexchange.com/questions/265765/story-man-purchases-plantation-on-planet-finds-unstoppable-infestation-uses)
*   [Laymen's description of "modals" to clients](https://ux.stackexchange.com/questions/143948/laymens-description-of-modals-to-clients)
*   [Was 36-bits needed for LISP?](https://retrocomputing.stackexchange.com/questions/24867/was-36-bits-needed-for-lisp)
*   [Alternatives to rim tape](https://bicycles.stackexchange.com/questions/84998/alternatives-to-rim-tape)

[Question feed](https://stackoverflow.com/feeds/question/24010109 "Feed of this question and its answers")