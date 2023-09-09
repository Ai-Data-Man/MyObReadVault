---
{"title":"java - Why is not possible to reopen a closed (standard) stream? - Stack Overflow","url":"https://stackoverflow.com/questions/33555283/why-is-not-possible-to-reopen-a-closed-standard-stream","clipped_at":"2022-06-10 19:50:01","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/java-Why-is-not-possible-to-reopen-a-closed-standard-stream-Stack-Overflow_1654861801/","dgPassFrontmatter":true}
---


# java - Why is not possible to reopen a closed (standard) stream? - Stack Overflow

# [Why is not possible to reopen a closed (standard) stream?](https://stackoverflow.com/questions/33555283/why-is-not-possible-to-reopen-a-closed-standard-stream)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 6 years, 7 months ago

Modified [6 years, 6 months ago](https://stackoverflow.com/questions/33555283/why-is-not-possible-to-reopen-a-closed-standard-stream?lastactivity "2015-11-17 14:50:40Z")

Viewed 16k times

31

4

[](https://stackoverflow.com/posts/33555283/timeline)

`System.in` is the "standard" input stream which supplies user input data. Once closed, this stream can not be re-opened. One such example is in the case of using a scanner to read the user input as follows:

```java
public class Test {
    public static void main(String[] args) {

        boolean finished;

        do {
            Scanner inputScanner = new Scanner(System.in);
            finished = inputScanner.hasNext("exit");
            boolean validNumber = inputScanner.hasNextDouble();
            if (validNumber) {
                double number = inputScanner.nextDouble();

                System.out.print(number);
            } else if (!finished) {
                System.out.println("Please try again.");
            }
            inputScanner.close();
        } while (!finished);
    }
}
```

In this example, an instance of type `Scanner` is created and used to read a series of numbers from the user (please ignore other details with this code which go beyond the scope of this example, I know the scanner should be created and closed outside the loop). After a number is retrieved from user input, the instance of this `Scanner` (i.e., the input stream) is closed. However, when another number is requested from user, and new instance is created, the input stream cannot be opened again. In case of this example, it creates a infinite loop.

The question is: why is not possible to reopen a closed stream?

[java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")

[Share](https://stackoverflow.com/q/33555283 "Short permalink to this question")

Follow

[edited Nov 6, 2015 at 0:51](https://stackoverflow.com/posts/33555283/revisions "show all edits to this post")

asked Nov 5, 2015 at 21:38

[

![user avatar](/img/user/阅读库藏/assets/1654861801-2a6ecf69b0963ba393f67af9ec7d047b.jpg)

](https://stackoverflow.com/users/4086836/andrei)

[Andrei](https://stackoverflow.com/users/4086836/andrei)

6,86366 gold badges3131 silver badges6060 bronze badges

*   Because its resources have already been released back to the operating system. Why aren't you opening the scanner and closing it outside the loop?
    
    – [RealSkeptic](https://stackoverflow.com/users/4125191/realskeptic "33,248 reputation")
    
    [Nov 5, 2015 at 21:46](#comment54890543_33555283)
    
*   I'll just link your [previous question](http://stackoverflow.com/questions/33552505/scanner-continuous-loop). Both questions aren't duplicates, because on the 1st one the question was "why my Scanner isn't blocked after each iteration". But there are some useful tips and explanations which might be useful for future readers about the question here asked "Why you cannot reopen a closed stream".
    
    – [Frakcool](https://stackoverflow.com/users/2180785/frakcool "10,540 reputation")
    
    [Nov 5, 2015 at 21:51](#comment54890698_33555283)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 4 Answers

Sorted by:

Highest score (default) Date modified (newest first) Date created (oldest first)

34

+50

[](https://stackoverflow.com/posts/33555444/timeline)

> why is not possible to reopen a closed stream in Java?

That's simply the nature of the underlying operating system constructs that Java streams represent. A stream is essentially a data conduit. Once you close it, it no longer exists. You may be able to create a new one between the same endpoints, but that yields a fundamentally different stream. We could go into implementation considerations such as buffering and stream positioning, but those are really side issues.

You also asked specifically about the standard streams. These are some of the cases that you cannot recreate. The operating system provides each process with its set of standard streams. Once they are closed, there is no way to obtain equivalents. You can put different streams in their place, but you cannot connect them to the original endpoints.

[Share](https://stackoverflow.com/a/33555444 "Short permalink to this answer")

Follow

answered Nov 5, 2015 at 21:49

[

![user avatar](/img/user/阅读库藏/assets/1654861801-195e0547aed59ca640c49b1b486e529d.png)

](https://stackoverflow.com/users/2402272/john-bollinger)

[John Bollinger](https://stackoverflow.com/users/2402272/john-bollinger)

138k88 gold badges7474 silver badges138138 bronze badges

*   I can agree with the fact that once you close a stream, it no longer exists. However, the user or the java process should have a dedicated stream that it can use or not ("dedicated" here is too heavy). Based on endpoints, the user should be able to recreate the stream. Seems reasonable enough, and it happens in typical client-server architecture. I can load a webpage, and glue my fingers to CTRL+R keys, and the server will still get accept new requests from clients. I have the endpoints, I should be able to recreate the stream. What am I missing?
    
    – [Andrei](https://stackoverflow.com/users/4086836/andrei "6,863 reputation")
    
    [Nov 6, 2015 at 0:48](#comment54894614_33555444)
    
*   2
    
    As I said, If you have the endpoints of a stream then you _may_ be able to create a replacement stream between those same endpoints. But with the standard streams, you know only one endpoint: your own program. Or take your web server example: the _server_ cannot reestablish a stream with a client once either end closes it. Even the client cannot be certain of doing so, for that matter. Suppose the server is part of a load-balanced server farm. If the client issues a new request then it might not go to the same physical server.
    
    – [John Bollinger](https://stackoverflow.com/users/2402272/john-bollinger "138,217 reputation")
    
    [Nov 6, 2015 at 13:58](#comment54914837_33555444)
    
*   More generally, any stream that is involved in IPC, which includes the standard streams much of the time, must be established cooperatively by the processes involved. Once such a stream is closed, by either side, it cannot be unilaterally recreated.
    
    – [John Bollinger](https://stackoverflow.com/users/2402272/john-bollinger "138,217 reputation")
    
    [Nov 6, 2015 at 14:04](#comment54915057_33555444)
    
*   1
    
    You may want to examine a Filter stream that maps the `close()` operation to a nop. As others have already answered, the stream itself is destroyed and not recoverable. The file that provided it may have been deleted when you closed it, the network connection ended.
    
    – [Norwæ](https://stackoverflow.com/users/2432128/norw%c3%a6 "1,515 reputation")
    
    [Nov 18, 2015 at 11:31](#comment55324264_33555444)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

13

[](https://stackoverflow.com/posts/33749378/timeline)

When you close the standard input stream:

*   If your input was being provided by a pipe, the other end of the pipe is notified. It will close its end and stop sending data. There is no way to tell it you made a mistake and it should start sending again;
    
*   If your input was being provided by a file, the OS drops its reference to the file and completely forgets that you were using it. There is just no way provided for you to reopen standard input and continue reading;
    
*   If your input was being provided by the console, it works with a pipe. The console is notified, will close its end of the pipe and stop sending you data.
    

So there's no way to reopen standard input.

BUT... there is also no reason to _close_ standard input, so just don't do that!

A good pattern to follow is:

*   The code or class that _opens_ a file is responsible for closing it.
    
*   If you pass an InputStream to another method that reads from it, that method should _not_ close it. Leave that to the code that opened it. It's like the streams _owner_.
    
*   Similarly, if you pass an OutputStream to another method that writes to it, that method should _not_ close it. Leave that to the code that owns it. BUT if you wrap the stream in other classes that may buffer some data _do_ call .flush() on them to make sure everything comes out!
    
*   If you're writing your own wrapper classes around InputStream and OutputStream, _don't_ close the delegate stream in your finalizer. If a stream needs to be cleaned up during GC, it should handle that itself.
    

In your example code, just don't close that Scanner. You didn't open standard input, so you shouldn't need to close it.

[Share](https://stackoverflow.com/a/33749378 "Short permalink to this answer")

Follow

answered Nov 17, 2015 at 4:48

[

![user avatar](/img/user/阅读库藏/assets/1654861801-d74a515038c9ceda2c4a7ad3f416f9aa.png)

](https://stackoverflow.com/users/5483526/matt-timmermans)

[Matt Timmermans](https://stackoverflow.com/users/5483526/matt-timmermans)

45.9k33 gold badges3535 silver badges7676 bronze badges

*   With re-open, I mean to create a stream. I understand from all the answers that it is not possible to re-open because you lose the initial references to the end points. However, if we look at the sample code above, every time the loop is executed, it should create a new stream. Think about your browser and an internet page. If you load that page once, what holds you back from refresing the page?
    
    – [Andrei](https://stackoverflow.com/users/4086836/andrei "6,863 reputation")
    
    [Nov 18, 2015 at 9:59](#comment55320291_33749378)
    
*   The difference is that your browser has an URL that tells it how to get at the page. Your process doesn't have any information about what its standard input file refers to. The underlying file is actually opened by the parent process (the one that creates yours), and you just get a handle to the open file. That's a a number that identifies the file in the OS. It's not a set of instructions for creating a stream, like an URL is.
    
    – [Matt Timmermans](https://stackoverflow.com/users/5483526/matt-timmermans "45,904 reputation")
    
    [Nov 18, 2015 at 13:37](#comment55329438_33749378)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

4

[](https://stackoverflow.com/posts/33749222/timeline)

Because Streams are unbounded. You peek values from streams as you need. Then when done simply close it. Streams does not hold it's all data in memory. Streams are designed to process relatively big amount of data which can't be held in memory. So you can't reopen an stream simply because you already have made a loop over it and exhausted all the data. As stream does not hold those data in memory. They are simply lost and that's why you can't reopen it. The better is you create a new stream than reopen an existing one.

[Share](https://stackoverflow.com/a/33749222 "Short permalink to this answer")

Follow

answered Nov 17, 2015 at 4:33

[

![user avatar](/img/user/阅读库藏/assets/1654861801-b53cf8359cd9b1b9eee7aa1d31b569db.png)

](https://stackoverflow.com/users/4807525/sohan-nohemy)

[sohan nohemy](https://stackoverflow.com/users/4807525/sohan-nohemy)

58544 silver badges1313 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

0

[](https://stackoverflow.com/posts/33691273/timeline)

Java standard library has chosen a "standardized" approach to `InputStream`. Even if you may legitimately perceive some streams, such as data incoming from the input console, as logically re-openable, the `InputStream` represents a generic approach, as it is intended to cover all the possible `InputStream`s, which many of them are by their nature not re-openable. As described perfectly in @JohnBollinger's answer.

[Share](https://stackoverflow.com/a/33691273 "Short permalink to this answer")

Follow

answered Nov 13, 2015 at 11:02

[

![user avatar](/img/user/阅读库藏/assets/1654861801-dabe1556077d7753292f212dd437d613.png)

](https://stackoverflow.com/users/2886891/honza-zidek)

[Honza Zidek](https://stackoverflow.com/users/2886891/honza-zidek)

8,11744 gold badges6363 silver badges9797 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

  

## Your Answer

### Sign up or [log in](https://stackoverflow.com/users/login?ssrc=question_page&returnurl=https%3a%2f%2fstackoverflow.com%2fquestions%2f33555283%2fwhy-is-not-possible-to-reopen-a-closed-standard-stream%23new-answer)

Sign up using Google

Sign up using Facebook

Sign up using Email and Password

 

### Post as a guest

Name

Email

Required, but never shown

Post Your Answer

By clicking “Post Your Answer”, you agree to our [terms of service](https://stackoverflow.com/legal/terms-of-service/public), [privacy policy](https://stackoverflow.com/legal/privacy-policy) and [cookie policy](https://stackoverflow.com/legal/cookie-policy)

## Not the answer you're looking for? Browse other questions tagged [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") or [ask your own question](https://stackoverflow.com/questions/ask).

The Overflow Blog

*   [The Great Decentralization? Geographic shifts and where tech talent is moving...](https://stackoverflow.blog/2022/06/08/the-great-decentralization-geographic-shifts-and-where-tech-talent-is-moving-next/?cb=1 "The Great Decentralization? Geographic shifts and where tech talent is moving next")
    
*   [Want to be great at UX research? Take a cue from cultural anthropology (Ep. 451)](https://stackoverflow.blog/2022/06/10/want-to-be-great-at-ux-research-take-a-cue-from-cultural-anthropology-ep-451/?cb=1)
    

Featured on Meta

*   [Announcing the arrival of Valued Associate #1214: Dalmarus](https://meta.stackexchange.com/questions/378862/announcing-the-arrival-of-valued-associate-1214-dalmarus?cb=1)
    
*   [Improvements to site status and incident communication](https://meta.stackexchange.com/questions/378941/improvements-to-site-status-and-incident-communication?cb=1)
    
*   [Collectives Update: Introducing Bulletins](https://meta.stackoverflow.com/questions/418333/collectives-update-introducing-bulletins?cb=1)
    
*   [The \[comma\] tag is being burninated](https://meta.stackoverflow.com/questions/405831/the-comma-tag-is-being-burninated?cb=1)
    
*   [Ask Wizard Test Results and Next Steps](https://meta.stackoverflow.com/questions/418491/ask-wizard-test-results-and-next-steps?cb=1)
    

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [Was it Gollum who untied Sam's Elf-rope?](https://scifi.stackexchange.com/questions/264145/was-it-gollum-who-untied-sams-elf-rope)
*   [String formatting in C++](https://codereview.stackexchange.com/questions/277203/string-formatting-in-c)
*   [Who were these characters?](https://scifi.stackexchange.com/questions/264166/who-were-these-characters)
*   [How to run scripts using python instead of python3?](https://askubuntu.com/questions/1413078/how-to-run-scripts-using-python-instead-of-python3)
*   [What tool should I use to remove this screw in my miter saw motor housing?](https://diy.stackexchange.com/questions/250787/what-tool-should-i-use-to-remove-this-screw-in-my-miter-saw-motor-housing)
*   [Ultegra RD-R8000 derailleur chewing up shifter cable - WTH?](https://bicycles.stackexchange.com/questions/84251/ultegra-rd-r8000-derailleur-chewing-up-shifter-cable-wth)
*   [Number of registrants for a virtual talk I organized turned out to be small, should I notify the speaker beforehand?](https://academia.stackexchange.com/questions/185926/number-of-registrants-for-a-virtual-talk-i-organized-turned-out-to-be-small-sho)
*   [Can't figure out the phrase "designer birdhouses" in this headline](https://ell.stackexchange.com/questions/317006/cant-figure-out-the-phrase-designer-birdhouses-in-this-headline)
*   [What is the difference between \`cat EOF\` and \`cat EOT\` and when should I use it?](https://unix.stackexchange.com/questions/705447/what-is-the-difference-between-cat-eof-and-cat-eot-and-when-should-i-use-it)
*   [How to Plot the solution of the heat equation on rectangle](https://mathematica.stackexchange.com/questions/269267/how-to-plot-the-solution-of-the-heat-equation-on-rectangle)
*   [Integration of Laplacian by parts](https://physics.stackexchange.com/questions/712935/integration-of-laplacian-by-parts)
*   [Translation of "Spare the rod and spoil the child."](https://chinese.stackexchange.com/questions/51473/translation-of-spare-the-rod-and-spoil-the-child)
*   [Constable of the river book](https://scifi.stackexchange.com/questions/264181/constable-of-the-river-book)
*   [A book about a spaceship with a forest interior](https://scifi.stackexchange.com/questions/264170/a-book-about-a-spaceship-with-a-forest-interior)
*   [Does buying and selling of bonds affect market price of bonds?](https://money.stackexchange.com/questions/151246/does-buying-and-selling-of-bonds-affect-market-price-of-bonds)
*   [Are protests in a democracy ethical?](https://philosophy.stackexchange.com/questions/91585/are-protests-in-a-democracy-ethical)
*   [Easy way to convert \\command{x}{y} to \\command{x,y}](https://tex.stackexchange.com/questions/647164/easy-way-to-convert-commandxy-to-commandx-y)
*   [Is printing money any different from taxing people?](https://politics.stackexchange.com/questions/73498/is-printing-money-any-different-from-taxing-people)
*   [Why don't we have a usable IP address in 127.0.0.0?](https://superuser.com/questions/1725375/why-dont-we-have-a-usable-ip-address-in-127-0-0-0)
*   [Why can't Ukraine ship grain from Mariupol?](https://politics.stackexchange.com/questions/73535/why-cant-ukraine-ship-grain-from-mariupol)
*   [Count /\[^a-z\]/ig with /\[a-z\]/ig](https://codegolf.stackexchange.com/questions/248352/count-a-z-ig-with-a-z-ig)
*   [Why does “&” look nothing like e and t](https://linguistics.stackexchange.com/questions/44620/why-does-look-nothing-like-e-and-t)
*   [Do I need PVC expansion coupling with a 130F temp range but very short run?](https://diy.stackexchange.com/questions/250833/do-i-need-pvc-expansion-coupling-with-a-130f-temp-range-but-very-short-run)
*   [What are the reflective structures above and to the rear of the engines on a VC-25?](https://aviation.stackexchange.com/questions/93510/what-are-the-reflective-structures-above-and-to-the-rear-of-the-engines-on-a-vc)

[Question feed](https://stackoverflow.com/feeds/question/33555283 "Feed of this question and its answers")