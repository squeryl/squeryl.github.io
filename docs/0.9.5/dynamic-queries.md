---
layout: manual
title: Dynamically varying Queries
headtitle: Dynamic Queries -
---

<a name='DynamicQuery'></a>

Dynamic Queries
---------------

It is often necessary to create queries that varies depending on values
known only at runtime.

### Dynamically excluding tables (or sub queries) from a Query

The method **inhibitWhen(b: Boolean)** available on
**org.squeryl.Queryable\[A\]**  
is meant to be called in a from clause, it will turn a Table\[A\] or
(sub) Query\[A\]  
into a Queryable\[Option\[A\]\] and remove the queryable from the
query  
**and** all expression referencing this queryable.

Example :

<script type="syntaxhighlighter" class="brush: scala">
<![CDATA[

def searchForBooks(authorLastName: Option[String], bookTitle: String) =
  from(authors.inhibitWhen(authorName  None), books)((a,b) =>
    where(a.get.lastName like authorLastName.get and b.title like bookTitle and a.get.id = b.authorId)
    select((b,a)))

val result1:List[(Book,Option(Author))] = searchForBooks(None, “Un Loup est un loup”).toList

val result2:List[(Book,Option(Author))] = searchForBooks(“Tolstoi”,“War and Peace”).toList
]]>

</script>

Notice the Option\[\] in the result type of the query, and how the Author
table disappears from the generated SQL base on the input of **inhibitWhen**:

— first query: searchForBooks(None, “Un Loup est un loup”).toList

<script type="syntaxhighlighter" class="brush: sql">
<![CDATA[

Select *
  from
  Book b
  where
  b.title like ? — ‘Un Loup est un loup’
]]>

— second query : searchForBooks(“Tolstoi”, “War and Peace”).toList

<script type="syntaxhighlighter" class="brush: sql">
<![CDATA[

Select *
  from
  Author a,
  Book b
  where
    a.lastName like ? and — “Tolstoi”
    b.title like ? and — “War and Peace”
    a.id = b.authorId
]]>

</script>

<a name='dyn-where-clause'></a>
{% include 0.9.5/feature-dyn-where-clause.html%}
