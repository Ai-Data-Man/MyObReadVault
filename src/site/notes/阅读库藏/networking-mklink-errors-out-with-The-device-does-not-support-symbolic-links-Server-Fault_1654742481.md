---
{"title":"networking - mklink errors out with \"The device does not support symbolic links\" - Server Fault","url":"https://serverfault.com/questions/797142/mklink-errors-out-with-the-device-does-not-support-symbolic-links","clipped_at":"2022-06-09 10:41:21","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/networking-mklink-errors-out-with-The-device-does-not-support-symbolic-links-Server-Fault_1654742481/","dgPassFrontmatter":true}
---


# networking - mklink errors out with "The device does not support symbolic links" - Server Fault

# [mklink errors out with "The device does not support symbolic links"](https://serverfault.com/questions/797142/mklink-errors-out-with-the-device-does-not-support-symbolic-links)

[Ask Question](https://serverfault.com/questions/ask)

Asked 5 years, 9 months ago

Modified [5 years, 9 months ago](https://serverfault.com/questions/797142/mklink-errors-out-with-the-device-does-not-support-symbolic-links?lastactivity "2016-08-18 00:23:20Z")

Viewed 8k times

5

[](https://serverfault.com/posts/797142/timeline)

I need to store symbolic links in a network folder. Trying this:

> mklink \\edgeserver\\public\\test.pdf \\fileserver1\\files\\test.pdf

and getting an error:

> The device does not support symbolic links

The command prompt is running on Windows Server 2008 R2 but I am not sure what underlying technology is used for network shares.

Is there a way to store symbolic links in a network share?

[networking](https://serverfault.com/questions/tagged/networking "show questions tagged 'networking'") [windows-server-2008](https://serverfault.com/questions/tagged/windows-server-2008 "show questions tagged 'windows-server-2008'") [symbolic-link](https://serverfault.com/questions/tagged/symbolic-link "show questions tagged 'symbolic-link'")

[Share](https://serverfault.com/q/797142 "Short permalink to this question")

[Improve this question](https://serverfault.com/posts/797142/edit)

Follow

asked Aug 16, 2016 at 14:44

[

![user avatar](/img/user/阅读库藏/assets/1654742481-f575462866ad09076a2687345cc1d35e.png)

](https://serverfault.com/users/187415/user1044169)

[user1044169](https://serverfault.com/users/187415/user1044169)

15111 gold badge22 silver badges33 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 1 Answer

Sorted by:

Highest score (default) Date modified (newest first) Date created (oldest first)

1

[](https://serverfault.com/posts/797150/timeline)

**EDIT**

I just noticed it looks a lot like you're trying to create a symlink between **two** network shares. It doesn't surprise me at all that this doesn't work properly. Why would you want to do such a thing? I got the same error when I tested just now too using Windows 10 and and two Linux Samba servers. I'm betting this functionality is just not supported, but if it is, perhaps it's only supported by Windows servers. I can't test that easily.

What OS do your storage servers run?

**Original answer**

According to this site, you may need to make some configuration changes for that to work. The site has better formatting, but I've quoted below in case the site goes away.

[http://sourcedaddy.com/windows-7/how-to-create-symbolic-links-to-shared-folders.html](http://sourcedaddy.com/windows-7/how-to-create-symbolic-links-to-shared-folders.html)

```plain
The mklink command also allows you to create a symbolic link targeting a Universal Naming Convention (UNC) path. For example, if you run the following command, Windows will create a symbolic link file called Link.txt that opens the Target.txt file.

Mklink link.txt \\server\folder\target.txt
If you enable remote symbolic links (discussed later in this section), they can be used to store symbolic links on shared folders and automatically redirect multiple Windows network clients to a different file on the network.

By default, you can use symbolic links only on local volumes. If you attempt to access a symbolic link located on a shared folder (regardless of the location of the target) or copy a symbolic link to a shared folder, you will receive an error. You can change this behavior by configuring the following Group Policy setting:

Computer Configuration\Administrative Templates\System\NTFS File System\Selectively Allow The Evaluation Of A SymbolicLink

When you enable this policy setting, you can select from four settings:

Local Link To Local Target Enabled by default, this allows local symbolic links to targets on the local file system.
Local Link To Remote Target Enabled by default, this allows local symbolic links to targets on shared folders.
Remote Link To Remote Target Disabled by default, this allows remote symbolic links to remote targets on shared folders.
Remote Link To Local Target Disabled by default, this allows remote symbolic links to remote targets on shared folders.
Enabling remote links can introduce security vulnerabilities. For example, a malicious user can create a symbolic link on a shared folder that references an absolute path on the local computer. When a user attempts to access the symbolic link, he will actually be accessing a different file that might contain confidential information. In this way, a sophisticated attacker might be able to trick a user into compromising the confidentiality of a file on his local computer.
```

Also I noticed you have a missing backslash in your UNC path, it should be `\\fileserver1\files\test.pdf`

[Share](https://serverfault.com/a/797150 "Short permalink to this answer")

[Improve this answer](https://serverfault.com/posts/797150/edit)

Follow

[edited Aug 18, 2016 at 0:23](https://serverfault.com/posts/797150/revisions "show all edits to this post")

answered Aug 16, 2016 at 14:58

[

![user avatar](/img/user/阅读库藏/assets/1654742481-813269d1b1f7162fa9aad409576ade29.jpg)

](https://serverfault.com/users/313604/ryan-babchishin)

[Ryan Babchishin](https://serverfault.com/users/313604/ryan-babchishin)

6,11022 gold badges1616 silver badges3636 bronze badges

*   I tried that but still getting the "The device does not support symbolic links" error. The error is only when the symbolic link itself resides on a network share. I can point a locally stored link to a file stored on a network without issues, but when the link itself is on the network, it doesn't work.
    
    – [user1044169](https://serverfault.com/users/187415/user1044169 "151 reputation")
    
    [Aug 16, 2016 at 16:10](#comment1009467_797150)
    
*   @user1044169 I wonder if it is not implemented on the file server's side. What is providing the file share? Another Windows server? A NetApp? A Linux server? Maybe it's documented somewhere.
    
    – [Ryan Babchishin](https://serverfault.com/users/313604/ryan-babchishin "6,110 reputation")
    
    [Aug 17, 2016 at 3:05](#comment1009631_797150)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

  

## Your Answer

### Sign up or [log in](https://serverfault.com/users/login?ssrc=question_page&returnurl=https%3a%2f%2fserverfault.com%2fquestions%2f797142%2fmklink-errors-out-with-the-device-does-not-support-symbolic-links%23new-answer)

Sign up using Google

Sign up using Facebook

Sign up using Email and Password

 

### Post as a guest

Name

Email

Required, but never shown

Post Your Answer

By clicking “Post Your Answer”, you agree to our [terms of service](https://stackoverflow.com/legal/terms-of-service/public), [privacy policy](https://stackoverflow.com/legal/privacy-policy) and [cookie policy](https://stackoverflow.com/legal/cookie-policy)

## Not the answer you're looking for? Browse other questions tagged [networking](https://serverfault.com/questions/tagged/networking "show questions tagged 'networking'") [windows-server-2008](https://serverfault.com/questions/tagged/windows-server-2008 "show questions tagged 'windows-server-2008'") [symbolic-link](https://serverfault.com/questions/tagged/symbolic-link "show questions tagged 'symbolic-link'") or [ask your own question](https://serverfault.com/questions/ask).

The Overflow Blog

*   [On the quantum internet, data doesn’t stream; it teleports (Ep. 450)](https://stackoverflow.blog/2022/06/07/on-the-quantum-internet-data-doesnt-steam-it-teleports/?cb=1)
    
*   [The Great Decentralization? Geographic shifts and where tech talent is moving...](https://stackoverflow.blog/2022/06/08/the-great-decentralization-geographic-shifts-and-where-tech-talent-is-moving-next/?cb=1 "The Great Decentralization? Geographic shifts and where tech talent is moving next")
    

Featured on Meta

*   [Announcing the arrival of Valued Associate #1214: Dalmarus](https://meta.stackexchange.com/questions/378862/announcing-the-arrival-of-valued-associate-1214-dalmarus?cb=1)
    
*   [Improvements to site status and incident communication](https://meta.stackexchange.com/questions/378941/improvements-to-site-status-and-incident-communication?cb=1)
    

#### Related

[

4

](https://serverfault.com/q/112334?rq=1 "Question score (upvotes - downvotes)")[Windows Backup to network share (Server 2008)](https://serverfault.com/questions/112334/windows-backup-to-network-share-server-2008?rq=1)

[

6

](https://serverfault.com/q/170964?rq=1 "Question score (upvotes - downvotes)")[Can't create or follow symlinks from linux client with a cifs mounted Windows Server 2008 R2 share](https://serverfault.com/questions/170964/cant-create-or-follow-symlinks-from-linux-client-with-a-cifs-mounted-windows-se?rq=1)

[

1

](https://serverfault.com/q/464047?rq=1 "Question score (upvotes - downvotes)")[Cygwin does not see Windows share as folder - no “d” attribute?](https://serverfault.com/questions/464047/cygwin-does-not-see-windows-share-as-folder-no-d-attribute?rq=1)

[

1

](https://serverfault.com/q/588110?rq=1 "Question score (upvotes - downvotes)")[mysql load\_file() function always return null for symbolic link](https://serverfault.com/questions/588110/mysql-load-file-function-always-return-null-for-symbolic-link?rq=1)

[

0

](https://serverfault.com/q/840389?rq=1 "Question score (upvotes - downvotes)")[SQL Server support for symbolic links](https://serverfault.com/questions/840389/sql-server-support-for-symbolic-links?rq=1)

[

11

](https://serverfault.com/q/957256?rq=1 "Question score (upvotes - downvotes)")[In Windows, is it possible to share a symlink as a network share?](https://serverfault.com/questions/957256/in-windows-is-it-possible-to-share-a-symlink-as-a-network-share?rq=1)

[

2

](https://serverfault.com/q/997701?rq=1 "Question score (upvotes - downvotes)")[Why am I getting different results using mklink and New-Item -ItemType SymbolicLink?](https://serverfault.com/questions/997701/why-am-i-getting-different-results-using-mklink-and-new-item-itemtype-symbolicl?rq=1)

[

0

](https://serverfault.com/q/1057994?rq=1 "Question score (upvotes - downvotes)")[Symbolic Link in Windows not behaving as expected](https://serverfault.com/questions/1057994/symbolic-link-in-windows-not-behaving-as-expected?rq=1)

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [Guess the secret word](https://puzzling.stackexchange.com/questions/116536/guess-the-secret-word)
*   [SF cyberpunk graphic novel from 80s involving a habitat called the hoop and its occupants hoopers](https://scifi.stackexchange.com/questions/264096/sf-cyberpunk-graphic-novel-from-80s-involving-a-habitat-called-the-hoop-and-its)
*   [How to multiply only a few columns of a table by some number in a compact way?](https://mathematica.stackexchange.com/questions/269183/how-to-multiply-only-a-few-columns-of-a-table-by-some-number-in-a-compact-way)
*   [Is Boseiju, who Endures' channel ability blocked by protection from green?](https://boardgames.stackexchange.com/questions/57505/is-boseiju-who-endures-channel-ability-blocked-by-protection-from-green)
*   [How to plot the following region with bold ribs?](https://mathematica.stackexchange.com/questions/269198/how-to-plot-the-following-region-with-bold-ribs)
*   [Multiple insert lines after vertical bar](https://tex.stackexchange.com/questions/647083/multiple-insert-lines-after-vertical-bar)
*   [How are these Covariant Derivative Identities found?](https://physics.stackexchange.com/questions/712683/how-are-these-covariant-derivative-identities-found)
*   [Photo Competition 2022-06-06: Reflections](https://photo.stackexchange.com/questions/129502/photo-competition-2022-06-06-reflections)
*   [Direction of fluid flow](https://engineering.stackexchange.com/questions/51264/direction-of-fluid-flow)
*   [Is there any consensus about what constitutes law (requirements for salvation) after Christ?](https://christianity.stackexchange.com/questions/91497/is-there-any-consensus-about-what-constitutes-law-requirements-for-salvation-a)
*   [What's inside this RF absorptive modulator?](https://electronics.stackexchange.com/questions/622736/whats-inside-this-rf-absorptive-modulator)
*   [Does my shifter needs to be replaced after crash damage?](https://bicycles.stackexchange.com/questions/84241/does-my-shifter-needs-to-be-replaced-after-crash-damage)
*   [Did the percent of humans living on < $2/day drop from 94% to 8.6% since 1820](https://skeptics.stackexchange.com/questions/53405/did-the-percent-of-humans-living-on-2-day-drop-from-94-to-8-6-since-1820)
*   [Is printing money any different from taxing people?](https://politics.stackexchange.com/questions/73498/is-printing-money-any-different-from-taxing-people)
*   [Are guns the number one killer of children in the US?](https://skeptics.stackexchange.com/questions/53399/are-guns-the-number-one-killer-of-children-in-the-us)
*   [How to make example.com work when \`www.\` is prepended at the beginning?](https://superuser.com/questions/1725347/how-to-make-example-com-work-when-www-is-prepended-at-the-beginning)
*   [is it rare to elide the final vowel if it is long?](https://latin.stackexchange.com/questions/18345/is-it-rare-to-elide-the-final-vowel-if-it-is-long)
*   [How to draw bracket trees in tikz?](https://tex.stackexchange.com/questions/646949/how-to-draw-bracket-trees-in-tikz)
*   [Why is exploitation necessary during training?](https://ai.stackexchange.com/questions/35827/why-is-exploitation-necessary-during-training)
*   [Feeling like a fraud at a startup I joined as an intern](https://workplace.stackexchange.com/questions/185410/feeling-like-a-fraud-at-a-startup-i-joined-as-an-intern)
*   [Activate a car's airbag with a kick](https://martialarts.stackexchange.com/questions/13337/activate-a-cars-airbag-with-a-kick)
*   [Tikz lines don't intersect nicely](https://tex.stackexchange.com/questions/647074/tikz-lines-dont-intersect-nicely)
*   [check unzip each archive to separate subdir as default](https://askubuntu.com/questions/1412907/check-unzip-each-archive-to-separate-subdir-as-default)
*   [Can you borrow foreign currency from local banks?](https://money.stackexchange.com/questions/151232/can-you-borrow-foreign-currency-from-local-banks)

[Question feed](https://serverfault.com/feeds/question/797142 "Feed of this question and its answers")