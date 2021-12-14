---
layout: manual
title: Evaluating arbitrary expressions on the database side 
---

The <u>&</u> function
---------------------

Squeryl’s select clause will take any closure in parameter and evaluate
it on the client side (client side as opposed to database side).
Sometimes the expressions needs to be evaluated on the database side,
this is what the **&** function does :

<script type="syntaxhighlighter" class="brush: scala">
<![CDATA[
// the * is done on the client side:
from(artists)(a => select(a.id * 1000))

// in this case it is computed by the database:
from(artists)(a => select(&(a.id * 1000)))
]]>
</script>

-   Note that the return type are equivalent in both cases, i.e. both
    queries resolve to **Query\[Float\]**

A select can have more than one invocation of & :

<script type="syntaxhighlighter" class="brush: scala">
<![CDATA[

val q: Query[Tuple2[Double, String]] = from(artists)(a => 
  select((&(a.id * 1000), &(a.firstName || a.lastName))))

]]>
</script>
