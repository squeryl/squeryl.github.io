---
layout: manual
title: Limitations
headtitle:
---

A query that returns a primitive type or a Tuple of primitive types (ex.: Query\[Int\], Query\[Int,String\]) **can not** be used as a subquery
----------------------------------------------------------------------------------------------------------------------------------------------

Nesting of such queries will result in an exception being thrown at
definition time with this error message :

*Sub query returns a primitive type or a Tuple of primitive type, and
therefore is not useable as a subquery in a from or join clause*

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

class Person(val firstName: val age: Int)

ASchema extends Schema {
  val people = table[Person]
}
]]>

</script>

The following queries cannot be used as subqueries, except in an **in**
clause :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

val peopleQuery1 = from(people)(p => select(p.age))

val peopleQuery2 = from(people)(p => select(p.age, p.name))

// NOT OK in a from clause:
from(peopleQuery1)(x => select(x))
from(peopleQuery2)(x => select(x._1))

// results in runtime error:
// Sub query returns a primitive type or a Tuple of primitive type, and therefore
// is not useable as a subquery in a from or join clause

// OK in an IN clause:
from(aTable)(t => t.aField in (peopleQuery1))
]]>

</script>

### Solution : Return the row object (the instance of A in table\[A\])

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

// this query is not nestable (in a from clause):
val nonNestablePeopleQuery1 = from(people)(p => select(p.age))

// this one is:
val nestablePeopleQuery1 = from(people)(p => select(p))

// therefore nesting in a ‘from’ is possible:
from(nestablePeopleQuery1)(x => select(x.age))

// unlike this one, which will fail at definition time:
from(nonNestablePeopleQuery1)(x => select(x))

// this kind of nesting is possible (provided of course that the
// type of aField is compatible with p.age):
from(t)(x => where(x.aField in (nonNestablePeopleQuery1)) select(x))
]]>

</script>

+, -, \* and / operators can cause an ambiguity in PrimitiveTypeMode
--------------------------------------------------------------------

<script type="syntaxhighlighter" class="brush: scala">


<![CDATA[

// Not OK:
val q1 = people.where(p=> p.age + 1 > 40)

// OK:
val q2 = people.where(p=> p.age plus 1 gt 40)
]]>

</script>
