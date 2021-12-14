---
layout: default
title: Frequently Asked Questions
headtitle: Frequently Asked Questions - 
---

### Q: Are relations supported ?

### A: Yes and No !

There is no concept in Squeryl of relations, but you can just as
easily  
implement them using the building blocks of the DSL, and end up  
with more reuse. Consider for example a “one to many” relation :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
class Song(val id, val title: String, val artistId: Long, val year) {
  def artist = artistsTable.where(a => a.id === artistId).single
}

class Artist(val id: Long, val name:String) {
  def songs = from(songsTable).where(s => s.artistId === id)
}
]]>

</script>

On each sides of the relation, you have to write one line of code,  
In JPA you would have an annotation on each side, the effort is  
slightly higher higher with Squeryl, but you get to reuse the code  
you wrote for the relation.

Suppose you want to filter the artists.songs relation, in JPA you  
would either filter the result on the client causing a waste of
resources,  
or define a separate query for filtering, loosing the opportunity for
reuse.

Since the **Artists.songs** method is a lazily evaluated Query\[Song\],
it can  
be further filtered with the ‘where’ method, yielding a unevaluated  
Query\[Song\] which will generate the SQL with the appropriate where
clause.

Here’s how it happens, suppose you execute the following :

<script type="syntaxhighlighter" class="brush: scala">


<![CDATA[

for(so <- jamesBrown.songs.where(s => s.year <= 1965))
  println(so.title)
]]>

</script>

The 1-M relation is filtered with a condition, and **only one**  
query is sent to the database, since jamesBrown.songs  
returns an unevaluated Query\[Song\], which becomes a subquery  
when .where(s.year \<= 1965) is called on it, and the combined  
query only get’s sent to the database for execution when the for
iteration begins.

To be fair with JPA, it must be said that not having the concept of  
relations means cascading deletes aren’t available. That is  
an area where Squeryl could be enhanced.
