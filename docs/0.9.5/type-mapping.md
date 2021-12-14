---
layout: manual
title: Mapping Types from SQL to Scala
headtitle: Mapping Types from SQL - 
---

Preserving Type Information
---------------------------

In order to be *type safe*, any SQL expression must be converted into a
Scala defined type, Squeryl does this using implicit type conversion
that are designed to ensure the following :

-   The bit length of a numeric expressions is equal to the bit length
    of the longest argument

<!-- -->

-   An expression results in an Option\[\] if at least one of it’s
    argument is of type Option\[\]

<!-- -->

-   The result of a numerical expression is floating point if it has
    least one floating point operation or argument

<!-- -->

-   All aggregate functions (avg, min, max, etc…) with the exception of
    count result are Option\[\]

In plain english these rule say that conversions are designed to not
cause loss of numerical precision,  
that floating point absorb (or dominate) as they do in mathematics, and
that the existence of an  
unknown (None) value in an expression, causes it’s result to become
unknown (None)

Here are some examples :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
val q1 = from(aTable)(t => select( &(t.aLong * t.aFloat) )): Query[Double]

val q2 = from(aTable)(t => compute( avg(t.aByte / t.anInt) )): Query[Option[Float]]

val q3 = from(aTable)(t => compute( t.aByte + count )): Query[Long]

val q4 = // || is the concatenation operator:
from(aTable)(t =>
  select( &(t.aString || t.aLong || " " || a.IntOption) )):
  Query[Option[String]]
]]>

</script>

Note how in query **q4** the presence of a single Option\[\] element
causes the result to become an Option\[\],  
this is consistent with the SQL92 standard. The **&** function is
explained [here](arbitrary-select-expressions.html) .
