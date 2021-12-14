---
layout: manual
title: <br/>
headtitle: Misc - 
---

Logging the generated SQL
-------------------------

The org.squeryl.Query.statement method returns the statement with the
JDBC’s prepared  
statement parameters substituted.

To log all SQL activity of a session, use
org.squeryl.Session.currentSession.setLogger(String =\> Unit)  
the closure receives all SQL statements and gets to do what it wants
with it.

Implicit conversions for single row aggregate queries
-----------------------------------------------------

In SQL, queries that consist of only aggregate functions (without group
by clause)  
always return exactly one row. Squeryl allows you to implicitly convert
them to tuples  
or to a scalar, as following example illustrates :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

val x1:Option[Float] = from(aTable)(t=> compute(avg(t.anInt)))

// or with an ascription:

val x2 = from(aTable)(t=> compute(avg(t.anInt))) : :Option[Float]

val t:(Option[Float],Option[String]) = from(aTable)(t=>
  compute(avg(t.anInt), min(t.aString)))
]]>

</script>

Instead of the (slightly) more verbose way :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

val x:Option[Float] = from(aTable)(t =>
  compute(avg(t.anInt))).single._1

val t:(Option[Float],Option[String]) = from(aTable)(t =>
  compute(avg(t.anInt), min(t.aString))).single
]]>

</script>

Is an object persisted ?
------------------------

The trait : **org.squeryl.PersistenceStatus** when mixed in a class
provides the **isPersisted** method, it is extended by KeyedEntity\[K\].
