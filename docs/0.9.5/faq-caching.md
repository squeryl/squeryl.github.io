---
layout: default
title: Frequently Asked Questions
headtitle: Frequently Asked Questions -
---

### Q: What kind of caching mechanism is supported ?

### A: No caching mechanism is supported at the moment.

However with Scala’s lazy val’s, a rudimentary (but useful) form of  
caching with lazy loading is possible :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

class Song(val id, val title: String, val artistId: Long) {
  lazy val artist = artists.where(a => a.id === artistId).single
}

]]>

</script>

In this case, aSong.artist would be fetched lazily and only once.
