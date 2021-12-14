---
layout: manual
title: Group and Aggregate Queries
headtitle: Group and Aggregate Queries - 
---

Group and Aggregate queries, unlike Select queries do not return
arbitrary types, but the types :  
Group, Measures and GroupWithMeasures that merely hold tuples.

Squeryl diverges slightly from SQL in that aggregate functions are
**not** allowed within a select.  
They are instead declared in a ‘compute’ clause which is in fact a
select in disguise, since  
it’s arguments end up in the generated SQL’s select clause. The
motivation for this design  
choice is to make it a bit harder to write invalid Select statements,
since the DSL forces  
a ‘compute’ clause to either replace a select or to follow a groupBy.

As the following example illustrates, the types of the resulting tuples
are determined by the  
arguments of the groupBy and compute clause.

| Aggregate Query Declaration                                                        | Query Type                                                                         |
|------------------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| from(aTable)(t=\> groupBy(t.aString,t.anInt))                                      | Query\[ **Group**\[(String,Int)\]\]                                                |
| from(aTable)(t=\> groupBy(t.aString,t.anIntOption))                                | Query\[ **Group**\[(String,Option\[Int\])\]\]                                      |
| from(aTable)(t=\> compute(min(t.aString),max(t.anInt)))                            | Query\[ **Measures**\[(Option\[String\],Option\[Int\])\]\]                         |
| from(aTable)(t=\> groupBy(t.aString,t.anInt) compute(max(t.aString),avg(t.anInt))) | Query\[ **GroupWithMeasures**\[(String,Int),(Option\[String\],Option\[Float\])\]\] |

groupBy and compute clauses are mutually exclusive with the select
clause in a query,  
in other words a query uses either select *or* a combination of groupBy
and compute.

Note how avg(t.anInt) transforms the return type into an
Option\[Float\].  
Rules for type conversions are explained in the [Type
Mapping](./miscellaneous.html) section.

The groupBy argument list is replicated in the SQL statement’s group by
clause  
and in the select, the compute argument list is appended to the select
list  
in the generated SQL.

Here is an example, the following Squeryl statement :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

def songCountByArtistId: Query[GroupWithMeasures[Long,Long]] =
  from(artists, songs)((a,s) =>
    where(a.id === s.artistId)
    groupBy(a.id)  
    compute(countDistinct(s.id)))
]]>

</script>

-   Note : countDistinct takes zero to many arguments, zero arguments
    translates into ‘count(distinct \*)’,

Translates into this SQL statement :

<script type="syntaxhighlighter" class="brush: sql">

<![CDATA[

Select
  Artist1.id as g0,
  count(distinct Song2.id) as c0
From
  Artist Artist1,
  Song Song2
Where
  (Artist1.id = Song2.artistId)
Group By
  Artist1.id
]]>

</script>

Notice how the groupBy(a.id) causes Artist1.id to be in the  
select list ***and*** in the group by clause, while compute(count)  
puts the aggregate function ‘count’ in the select list.
