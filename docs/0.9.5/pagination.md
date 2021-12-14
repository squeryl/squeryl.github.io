---
layout: manual
title: Result Set Pagination
headtitle: Pagination - 
---

Paginated queries are supported via the **page(offset:Int,
pageLength:Int)** method on all org.squeryl.Query\[A\]

Example :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

def searchForBooks(title: String, offset: Int, pageLength: Int) =
  from(books)(b =>
    where(b.title like title)
    select(b)
    orderBy(b.title asc)
  ).page(offset, pageLength)

val pageLength = 10

val page1 = searchForBooks(“The Art of%”, pageLength * 0, pageLength)

val page2 = searchForBooks(“The Art of%”, pageLength * 1, pageLength)

val page3 = searchForBooks(“The Art of%”, pageLength * 2, pageLength)
]]>

</script>

Squeryl’s DatabaseAdaptor uses the appropriate mechanism from each
database to paginate efficiently.
