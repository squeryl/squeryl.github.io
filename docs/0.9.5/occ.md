---
layout: manual
title: Optimistic Concurrency Control
headtitle: Optimistic Concurrency Control - 
---

The org.squeryl.Optimistic trait
--------------------------------

Classes that extend the Optimistic trait will have an optimistic
concurrency control policy enforced.

The Optimistic trait has the Int field **occVersionNumber** that gets
incremented on each update, when updating via table\[T\].update(t:T), a
StaleUpdateException is thrown if the row has been updated since ‘t’ was
fetched (i.e. if the occVersionNumber does not match the value it was
fetched with).

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

class Book(val id: Long,
  var title: String,
  var authorId: Long) extends KeyedEntity[Long] with Optimistic
]]>

</script>

As can be seen below the org.squeryl.Optimistic trait has a self type
declaration of KeyedEntity\[\], which means that a class must extend
KeyedEntity\[\] in order extend Optimistic.

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
trait Optimistic {
  self: KeyedEntity[_] =>
    protected val occVersionNumber = 0
}
]]>

</script>

### Forcing updates

The Table\[A\] class have the methods **forceUpdate(a:A)** and
**forceUpdate(a: Iterable\[A\])** to bypass the OCC mechanism
