---
{"title":"java - jOOQ Subquery in Order By - Stack Overflow","url":"https://stackoverflow.com/questions/61525633/jooq-subquery-in-order-by","clipped_at":"2022-05-29 15:02:23","tags":["无"],"dg-publish":true,"permalink":"/阅读库藏/java-jOOQ-Subquery-in-Order-By-Stack-Overflow_1653807743/","dgPassFrontmatter":true}
---


# java - jOOQ Subquery in Order By - Stack Overflow

# [jOOQ Subquery in Order By](https://stackoverflow.com/questions/61525633/jooq-subquery-in-order-by)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 2 years ago

Modified [2 years ago](https://stackoverflow.com/questions/61525633/jooq-subquery-in-order-by?lastactivity "2020-04-30 15:49:39Z")

Viewed 618 times

1

[](https://stackoverflow.com/posts/61525633/timeline)

I'm using a subquery in order by like this on MySQL 8 database:

```sql
select * from series 
order by (select max(competition.competition_date) from competition 
          where competition.series_id = series.id) desc
```

But I didn't find a way to do that with jOOQ.

I tried the following query but this does not compile:

```scss
dsl
   .selectFrom(SERIES)
   .orderBy(dsl.select(DSL.max(COMPETITION.COMPETITION_DATE))
               .from(COMPETITION).where(COMPETITION.SERIES_ID.eq(SERIES.ID)).desc())
   .fetch()
```

Are subqueries not supported in order by?

[java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [mysql](https://stackoverflow.com/questions/tagged/mysql "show questions tagged 'mysql'") [jooq](https://stackoverflow.com/questions/tagged/jooq "show questions tagged 'jooq'")

[Share](https://stackoverflow.com/q/61525633 "Short permalink to this question")

Follow

[edited Apr 30, 2020 at 14:59](https://stackoverflow.com/posts/61525633/revisions "show all edits to this post")

asked Apr 30, 2020 at 14:23

[

![user avatar](assets/1653807743-7e3aaa61b0506299f74ee6a161a4f651.jpg)

](https://stackoverflow.com/users/1045142/simon-martinelli)

[Simon Martinelli](https://stackoverflow.com/users/1045142/simon-martinelli)

27.7k33 gold badges3737 silver badges6565 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 2 Answers

Sorted by:

Highest score (default) Date modified (newest first) Date created (oldest first)

2

[](https://stackoverflow.com/posts/61527459/timeline)

### `Select<R> extends Field<R>`

There's a pending feature request [#4828](https://github.com/jOOQ/jOOQ/issues/4828) to let `Select<R> extend Field<R>`. This seems tempting because jOOQ already supports nested records to some extent for those dialects that support it.

But I have some doubts whether this is really a good idea in this case, because no database I'm aware of (i.e. where I tried this) supports scalar subqueries that project more than one column. It's possible to use such subqueries in row value expression predicates, e.g.

```sql
(a, b) IN (SELECT x, y FROM t)
```

But that's a different story, because it's limited to predicates, and not arbitrary column expressions. And it is already supported in jOOQ, via the various [`DSL.row()`](https://www.jooq.org/javadoc/latest/org.jooq/org/jooq/impl/DSL.html#row(org.jooq.Field,org.jooq.Field)) overloads, e.g.

```java
row(A, B).in(select(T.X, T.Y).from(T))
```

### `Select<Record1<T>> extends Field<T>`

This is definitely desireable, because a `SELECT` statement that projects only one column of type `T` really _is_ a `Field<T>` in SQL, i.e. a scalar subquery. But letting `Select<Record1<T>>` extend `Field<T>` is not possible in Java. There is no way to express this using Java's generics. If we wanted to do this, we'd have to "overload" the `Select` type itself and create

*   `Select1<T1> extends Select<Record1<T1>>`
*   `Select2<T1, T2> extends Select<Record2<T1, T2>>`
*   etc.

In that case, `Select1<T1>` could be a special case, extending `Field<T1>`, and the other ones would not participate in such a type hierarchy. But in order to achieve this, we'd have to duplicate the entire `Select` DSL API _per projection degree_, i.e. copy it 22 times, which is probably not worth it. There are already [67 `Select.*Step` types in the jOOQ API](https://www.jooq.org/javadoc/latest/org.jooq/org/jooq/package-summary.html), as of jOOQ 3.13. This makes it difficult to justify the enhancement even only for scalar subqueries, i.e. for `Select1`.

### Using [`DSL.field(Select<Record1<T>>)`](https://www.jooq.org/javadoc/latest/org.jooq/org/jooq/impl/DSL.html#field(org.jooq.Select)) and related API

[You've already found the right answer](https://stackoverflow.com/a/61526221/521799). While `Select<Record1<T>>` cannot extend `Field<T>`, we can accept `Select<? extends Record1<T>>` in plenty of API, as an overload to the usual `T|Field<T>` overloads. This has been done occasionally, and might be done more thoroughly throughout the API: [https://github.com/jOOQ/jOOQ/issues/7240](https://github.com/jOOQ/jOOQ/issues/7240).

It wouldn't help you, because you want to call `.desc()` on a column expression (the `Select`), rather than wrap pass it to a method, so we're back at Java's limitation mentioned before.

### Kotlin and other languages

If you're using Kotlin or other languages that have some way of providing "extension functions", however, you could use this approach:

```kotlin
inline fun <T> Select<Record1<T>>.desc(): SortField<T> {
    return DSL.field(this).desc();
}
```

jOOQ might provide these out of the box in the future: [https://github.com/jOOQ/jOOQ/issues/6256](https://github.com/jOOQ/jOOQ/issues/6256)

[Share](https://stackoverflow.com/a/61527459 "Short permalink to this answer")

Follow

answered Apr 30, 2020 at 15:49

[

![user avatar](assets/1653807743-04a85944492947abce730860ed76a871.jpg)

](https://stackoverflow.com/users/521799/lukas-eder)

[Lukas Eder](https://stackoverflow.com/users/521799/lukas-eder)

196k123123 gold badges646646 silver badges14061406 bronze badges

*   Thanks for the for the background
    
    – [Simon Martinelli](https://stackoverflow.com/users/1045142/simon-martinelli "27,670 reputation")
    
    [Apr 30, 2020 at 16:37](#comment108838772_61527459)
    

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

1

[](https://stackoverflow.com/posts/61526221/timeline)

Turning the subquery into a Field works:

```scss
dsl.selectFrom(SERIES)
   .orderBy(DSL.field(dsl.select(DSL.max(COMPETITION.COMPETITION_DATE)).from(COMPETITION)
                  .where(COMPETITION.SERIES_ID.eq(SERIES.ID))).desc())
   .fetch()
```

[Share](https://stackoverflow.com/a/61526221 "Short permalink to this answer")

Follow

answered Apr 30, 2020 at 14:49

[

![user avatar](assets/1653807743-7e3aaa61b0506299f74ee6a161a4f651.jpg)

](https://stackoverflow.com/users/1045142/simon-martinelli)

[Simon Martinelli](https://stackoverflow.com/users/1045142/simon-martinelli)

27.7k33 gold badges3737 silver badges6565 bronze badges

[Add a comment](# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

  

## Your Answer

### Sign up or [log in](https://stackoverflow.com/users/login?ssrc=question_page&returnurl=https%3a%2f%2fstackoverflow.com%2fquestions%2f61525633%2fjooq-subquery-in-order-by%23new-answer)

Sign up using Google

Sign up using Facebook

Sign up using Email and Password

 

### Post as a guest

Name

Email

Required, but never shown

Post Your Answer

By clicking “Post Your Answer”, you agree to our [terms of service](https://stackoverflow.com/legal/terms-of-service/public), [privacy policy](https://stackoverflow.com/legal/privacy-policy) and [cookie policy](https://stackoverflow.com/legal/cookie-policy)

## Not the answer you're looking for? Browse other questions tagged [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'") [mysql](https://stackoverflow.com/questions/tagged/mysql "show questions tagged 'mysql'") [jooq](https://stackoverflow.com/questions/tagged/jooq "show questions tagged 'jooq'") or [ask your own question](https://stackoverflow.com/questions/ask).

The Overflow Blog

*   [The complete beginners guide to graph theory](https://stackoverflow.blog/2022/05/26/the-complete-beginners-guide-to-graph-theory/?cb=1)
    
*   [Games are good, mods are immortal (ep 446)](https://stackoverflow.blog/2022/05/27/games-are-good-mods-are-immortal-ep-446/?cb=1)
    

Featured on Meta

*   [Announcing the arrival of Valued Associate #1214: Dalmarus](https://meta.stackexchange.com/questions/378862/announcing-the-arrival-of-valued-associate-1214-dalmarus?cb=1)
    
*   [Improvements to site status and incident communication](https://meta.stackexchange.com/questions/378941/improvements-to-site-status-and-incident-communication?cb=1)
    
*   [Staging Ground: Reviewer Motivation, Scaling, and Open Questions](https://meta.stackoverflow.com/questions/417932/staging-ground-reviewer-motivation-scaling-and-open-questions?cb=1)
    
*   [Collectives Update: Introducing Bulletins](https://meta.stackoverflow.com/questions/418333/collectives-update-introducing-bulletins?cb=1)
    
*   [Should we burninate the \[social\] tag?](https://meta.stackoverflow.com/questions/410740/should-we-burninate-the-social-tag?cb=1)
    
*   [Retiring Our Community-Specific Closure Reasons for Server Fault and Super User](https://meta.stackoverflow.com/questions/418095/retiring-our-community-specific-closure-reasons-for-server-fault-and-super-user?cb=1)
    

#### Related

[

474

](https://stackoverflow.com/q/3002605?rq=1 "Question score (upvotes - downvotes)")[How do I add indexes to MySQL tables?](https://stackoverflow.com/questions/3002605/how-do-i-add-indexes-to-mysql-tables?rq=1)

[

420

](https://stackoverflow.com/q/11990708?rq=1 "Question score (upvotes - downvotes)")[error: 'Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)' -- Missing /var/run/mysqld/mysqld.sock](https://stackoverflow.com/questions/11990708/error-cant-connect-to-local-mysql-server-through-socket-var-run-mysqld-mysq?rq=1)

[

282

](https://stackoverflow.com/q/14770671?rq=1 "Question score (upvotes - downvotes)")[MySQL order by before group by](https://stackoverflow.com/questions/14770671/mysql-order-by-before-group-by?rq=1)

[

2

](https://stackoverflow.com/q/28280039?rq=1 "Question score (upvotes - downvotes)")[jooq - execute string as subquery](https://stackoverflow.com/questions/28280039/jooq-execute-string-as-subquery?rq=1)

[

2

](https://stackoverflow.com/q/30192885?rq=1 "Question score (upvotes - downvotes)")[How to sort data in mysql UNION subquery?](https://stackoverflow.com/questions/30192885/how-to-sort-data-in-mysql-union-subquery?rq=1)

[

1

](https://stackoverflow.com/q/36379486?rq=1 "Question score (upvotes - downvotes)")[How to use jOOQ converter when using jOOQ to generate SQL without Code-genreation?](https://stackoverflow.com/questions/36379486/how-to-use-jooq-converter-when-using-jooq-to-generate-sql-without-code-genreatio?rq=1)

[

1

](https://stackoverflow.com/q/57501182?rq=1 "Question score (upvotes - downvotes)")[Avoiding fetch in Jooq subquery](https://stackoverflow.com/questions/57501182/avoiding-fetch-in-jooq-subquery?rq=1)

[

4

](https://stackoverflow.com/q/57506855?rq=1 "Question score (upvotes - downvotes)")[jOOQ Query OrderBy as String](https://stackoverflow.com/questions/57506855/jooq-query-orderby-as-string?rq=1)

[

1

](https://stackoverflow.com/q/63175409?rq=1 "Question score (upvotes - downvotes)")[How to access columns of subqueries with jooq?](https://stackoverflow.com/questions/63175409/how-to-access-columns-of-subqueries-with-jooq?rq=1)

#### [Hot Network Questions](https://stackexchange.com/questions?tab=hot)

*   [Recklessly endangering the English public by throwing objects](https://law.stackexchange.com/questions/80593/recklessly-endangering-the-english-public-by-throwing-objects)
*   [How do I tell a professor I don't find their research topic interesting?](https://academia.stackexchange.com/questions/185598/how-do-i-tell-a-professor-i-dont-find-their-research-topic-interesting)
*   [How long would it take for a Roman farmer to learn about Julius Caesars assassination?](https://worldbuilding.stackexchange.com/questions/230560/how-long-would-it-take-for-a-roman-farmer-to-learn-about-julius-caesars-assassin)
*   [Installing a newer version of mutt on quite old Debian](https://unix.stackexchange.com/questions/704207/installing-a-newer-version-of-mutt-on-quite-old-debian)
*   [How do trade languages form?](https://conlang.stackexchange.com/questions/1592/how-do-trade-languages-form)
*   [What's a reasonable Green Hag Coven encounter for a party of 3 lv 8 PCs?](https://rpg.stackexchange.com/questions/198715/whats-a-reasonable-green-hag-coven-encounter-for-a-party-of-3-lv-8-pcs)
*   [My native language is English but I was raised in a different country, how do I prove my native language?](https://academia.stackexchange.com/questions/185657/my-native-language-is-english-but-i-was-raised-in-a-different-country-how-do-i)
*   [I need the name or number of the bricks that the minifigures are on](https://bricks.stackexchange.com/questions/17149/i-need-the-name-or-number-of-the-bricks-that-the-minifigures-are-on)
*   [What do you do when an opponent asks for a score sheet after a game in which recording is not compulsory?](https://chess.stackexchange.com/questions/39884/what-do-you-do-when-an-opponent-asks-for-a-score-sheet-after-a-game-in-which-rec)
*   [How to do TDD in real world applications?](https://softwareengineering.stackexchange.com/questions/438897/how-to-do-tdd-in-real-world-applications)
*   [Finding zeros of a complex Airy function](https://mathematica.stackexchange.com/questions/268809/finding-zeros-of-a-complex-airy-function)
*   [Validating and storing credit card data for retrieval later](https://security.stackexchange.com/questions/262274/validating-and-storing-credit-card-data-for-retrieval-later)
*   [Why is it impossible for the reactor of the nuclear power plant to turn into an explosive nuclear bomb?](https://physics.stackexchange.com/questions/710668/why-is-it-impossible-for-the-reactor-of-the-nuclear-power-plant-to-turn-into-an)
*   [Why does the USA seem to be more crime infested than other OECD countries?](https://politics.stackexchange.com/questions/73346/why-does-the-usa-seem-to-be-more-crime-infested-than-other-oecd-countries)
*   [What motivated the weird boolean instruction repertoire of the PDP-11?](https://retrocomputing.stackexchange.com/questions/24557/what-motivated-the-weird-boolean-instruction-repertoire-of-the-pdp-11)
*   [How do I politely tell my former manager I no longer work for her?](https://workplace.stackexchange.com/questions/185194/how-do-i-politely-tell-my-former-manager-i-no-longer-work-for-her)
*   [unable to delete file as root](https://unix.stackexchange.com/questions/704069/unable-to-delete-file-as-root)
*   [What are the Roles of "quo" and "eo" In "the more...the more" Constructions: For Example: "The more you have the meaner you become!"?](https://latin.stackexchange.com/questions/18292/what-are-the-roles-of-quo-and-eo-in-the-more-the-more-constructions-for)
*   [Calculate the Lowest Even-Harmonic of the Values in a List](https://codegolf.stackexchange.com/questions/247927/calculate-the-lowest-even-harmonic-of-the-values-in-a-list)
*   [How fast do carbon and aluminium rim brake tracks wear?](https://bicycles.stackexchange.com/questions/84114/how-fast-do-carbon-and-aluminium-rim-brake-tracks-wear)
*   [How do I make a non-permanent death more impactful?](https://rpg.stackexchange.com/questions/198671/how-do-i-make-a-non-permanent-death-more-impactful)
*   [Should I be honest about a counteroffer to my prospective new employer?](https://workplace.stackexchange.com/questions/185178/should-i-be-honest-about-a-counteroffer-to-my-prospective-new-employer)
*   [Robot ants that stay forever on a rod](https://puzzling.stackexchange.com/questions/116351/robot-ants-that-stay-forever-on-a-rod)
*   [How to best take notes as a player in an ongoing campaign?](https://rpg.stackexchange.com/questions/198657/how-to-best-take-notes-as-a-player-in-an-ongoing-campaign)

[Question feed](https://stackoverflow.com/feeds/question/61525633 "Feed of this question and its answers")