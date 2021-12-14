---
layout: manual
title: Introduction
headtitle: Introduction -
---

### With all the Java ORMs available, why was Squeryl written ?

Squeryl is a departure from standard Java ORMs, we hope that the
following arguments will provide sufficient justification for not
following the Java standards, all of the arguments presented apply
equally to JPA, JDO or Hibernate.

JPA was designed to provide the maximum functionality and expressiveness
within the constraints of the Java language. All of them escape the
language with a string based query language (JPA-QL, JDOQL, HQL, etc…)
for situations where their imperative APIs aren’t sufficient or yields
poor performance. Squeryl does everything with a single language : Scala

Squeryl will have achieved something if one *never* has to define a
database a query as a raw string.

### HQL, JDOQL and JPA-QL queries are merely strings, and are therefore fragile

When a project has tens of thousands of lines of code, making changes to
the ORM’s schema requires extensive impact analysis, one must browse
through all string (whatever-QL) queries and verify if they are
affected. With Squeryl statements, the compiler will list all breaking
changes, and an IDEs can help you with the refactoring.

Here is a comparison of a Squeryl and a JPA version of the same query :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
//All is validated at compile time here:

var avg: Option[Float] = // The compiler ‘knows’ that this query returns an Option[Float]
  from(grades)(g =>
    // mathId has to by type compatible with g.subjectId to compile
    where(g.subjectId === mathId)
    compute(avg(g.scoreInPercentage)))
]]>

</script>
<!--
// partial select :
var name =
  from(students)(s =>
    where(s.id === idOfStudent)
    select(s.firstName)
  )
-->

The equivalent JPA query invocation is subject to a runtime failure at
five places :

<script type="syntaxhighlighter" class="brush: java">

<![CDATA[
// Equivalent JPA query
Query q = entityManager.createQuery(
  //We’ll get an SQLException if there’s a typo here:
  “SELECT AVG (g.scoreInPercentage) FROM Grades g where g.subjectId = :subjectId”);

// a runtime exception if mathId is of the wrong type
q.setParameter(1, mathId); // or if 1 is not the right index

// ClassCastExeption possible
Number avg = (Number) q.getSingleResult();

// NullPointerException if the query returns null
//(ex.: if there are no math Grades in the table)
avg.floatValue();
]]>

</script>

### String/annotation based queries inhibit reusability by not being composable

Reusability of code is a very desirable property. Many design patterns
are beneficial because they increase reuse. The current generation of
Java ORMs by using string expressions, annotations and XML don’t support
composition of queries, making reuse difficult. A Squeryl query on the
other hand, can be queried against as if it was a view or a table, so
they only need to be defined once and can be reused as sub queries.

### Current Java ORMs wrap a high level language (SQL) into a lower level API

A JPA style ORM expose an object model on which user code navigates from
one entity to another via relations that are represented as collections.
While this approach is adequate for simple cases it can lead to verbose
imperative code and excessive round trips to the database, which is why
they all have a (string based) query language (HQL, JPA-QL, etc..)  
<a name='squeryl-control-granularity-more'></a>  
{% include 0.9.5/squeryl-control-granularity-intro.html %} 
{% include 0.9.5/squeryl-control-granularity-more.html %}
