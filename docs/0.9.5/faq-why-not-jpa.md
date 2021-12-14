---
layout: default
title: Frequently Asked Questions
headtitle: Frequently Asked Questions - 
---

### Q: There is already a plethora of Java ORMs, do we *really* need one more ? What is wrong with JPA ?

### A: No Java ORMs JPA included, gives you both type safety and performance, you are forced to sacrifice one or the other

Can you spot the typo in this JPA query ?

<script type="syntaxhighlighter" class="brush: java">

<![CDATA[

//JPA:
Query query = em.createQuery(
  “select a,b,p from authors, books, publisher ” +
  “where b.subjectId = :idOfSubject and ” +
  " a.id = b.authorId and " +
  " b.publiserId = p.id "  
);

query.setParameter(“idOfSubject”, idOfSubject);
]]>

</script>

**Let’s hope that you can, because the compiler won’t tell you !**

If you happen to rename a column, change it’s type, remove it, etc. 
the compiler will be of no help.

If you use JPAs relation mechanism to *navigate* through the object graph,
like the following example illustrates, you will regain type safety,
but at the cost of horrible performance,
Can you count the number of database calls resulting from this JPA query?

<script type="syntaxhighlighter" class="brush: java">

<![CDATA[

Iterable<Book> books = bookEntityManager.findBySubjectId(idOfSubject);
while(books.hasNext()) {
  Book b = books.next();
  Author a = b.getAuthor();
  Publisher p = b.getPublisher();
  System.out.println(b.getTitle() + " by " + a.getFullName() + " published by " + p.getName());
}
]]>

</script>

The answer is : two database round trips for every row returned by the findBySubjectId query,
an N-ary relation typically causes O (N-1) calls to the database, which can have disastrous
performance consequences.

Squeryl allows you to combine the efficiency of the first approach
(the query is done in a single database call), and the type safety of
the second:

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

val q =
  from(authors, books, publisher)((a,b,p) =>
    where(b.subjectId = idOfSubject and
          a.id = b.authorId and
          b.publisherId === p.id)
   select((a,b,p)))

for(r <- q)
  println(r._2.title + " by " + r._1.fullName + " published by " + p._3.name)
]]>

</script>

<br/>

Squeryl’s main building block, the composable Query\[A\], coupled with
the laziness of Scala,  
allows the implementation of relations that compare advantageously with
JPAs  
you can read about it [here.](faq-relations.html)
