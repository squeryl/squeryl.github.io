---
layout: manual
title: Joins
headtitle: Joins - 
---

Joining with **from**
---------------------

A two table join :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

class Playlist(var id: Long, var name: String, var path: String) extends KeyedEntity[Long] {

  import MusicDb._

  // a two table join:
  def songsInPlaylistOrder =
    from(playlistElements, songs)((ple, s) =>
    where(ple.playlistId = id and ple.songId = s.id)
    select(s)
    orderBy(ple.songNumber asc))
}
]]>

</script>

The select clause can return anything that is in the scope, for example it  
could return both joined objects with a Tuple2 : select((s, ple))

The **join** keyword
--------------------

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

val ratingsForAllSongs =
  join(songs, ratings.leftOuter)((s,r) =>
    select(s, r)
   on(s.id === r.map(_.songId)))

for(sr <- ratingsForAllSongs)
  println(sr._1.title + " rating is " +
    sr._2.map(r => r.appreciationScore.toString).getOrElse(“not rated”))
]]>

</script>

produces :

<script type="syntaxhighlighter" class="brush: sql">

<![CDATA[

Select
  Song1.id as Song1_id,
  Rating2.songId as Rating2_songId,
  Rating2.songId as Rating2_songId,
  Rating2.appreciationScore as Rating2_appreciationScore,
  Rating2.userId as Rating2_userId,
  Song1.year as Song1_year,
  Song1.title as Song1_title,
  Song1.filePath as Song1_filePath,
  Song1.artistId as Song1_artistId,
  Song1.id as Song1_id
From
  Song Song1
left outer join Rating as Rating2 on (Song1.id = Rating2.songId)
]]>

</script>

A join can also be an agreate query, for example :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

val q =
  join(artists,songs.leftOuter)((a,s)=>
    groupBy(a.id, a.firstName)
    compute(countDistinct(s.map(_.id)))
    on(a.id === s.map(_.authorId)))
]]>

</script>

The type of this query is
**Query\[GroupWithMeasures\[(Long,String),Long\]**, it produces the
following SQL:

<script type="syntaxhighlighter" class="brush: sql">

<![CDATA[

Select
  Person1.id as g0,
  Person1.firstName as g1,
  count(distinct Song2.id) as c0
From
  Person Person1
left outer join Song as Song2 on (Person1.id = Song2.authorId)
Group By
  Person1.id,
  Person1.firstName
]]>

</script>

Of course, like in SQL, a **join** can have a where clause :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

val q =
  join(artists,songs.leftOuter)((a,s)=>
    where(a.id in myListOfArtistId)
    groupBy(a.id, a.firstName)
    compute(countDistinct(s.map(_.id)))
    on(a.id === s.map(\_.authorId)))
]]>

</script>

If a join has N arguments, the ‘on’ function must take N-1 arguments,  
the i’th ‘on’ condition corresponds to the i’th table expression :

<script type="syntaxhighlighter" class="brush: scala">


<![CDATA[

join(T, A1, A2,… AN)((a1,a2,…,aN) => …
  on(…condition for a1…,…condition for a2…,……condition for aN…, ))
]]>

</script>

Outer Joins *<span class="syntax deprecated"></span>*
-----------------------------------------------------

A Left Outer Join is done by calling leftOuterJoin within the select
clause,  
where the fist parameter is the row object variable, and the second a
boolean clause.

This query returns the joined objects in a tuple, leftOuterJoin causes
the second tuple  
member to be an Option\[Rating\]

<script type="syntaxhighlighter" class="brush: scala">


<![CDATA[

val ratingsForAllSongs =
  from(songs, ratings)((s,r) =>
    select((s, leftOuterJoin(r, s.id === r.songId))))

for(sr <- ratingsForAllSongs)
  println(sr._1.title + " rating is " +
    sr._2.map(r => r.appreciationScore.toString).getOrElse(“not rated”))
]]>

</script>

The signatures of (left, right and full) outer joins ensure that the
null part of the  
outer join returns an Option\[\]

leftOuterJoin\[A\](a: A, b: =\>LogicalBoolean): Option\[A\]

rightOuterJoin\[A,B\](a: A, b: B, c: =\>LogicalBoolean): (Option\[A\],
B)

fullOuterJoin\[A,B\](a: A, b:B, c: =\>LogicalBoolean): (Option\[A\],
Option\[B\])

The SQL of the previous query is :

<script type="syntaxhighlighter" class="brush: sql">

<![CDATA[

Select
  Song1.id as Song1_id,
  Rating2.songId as Rating2_songId,
  Rating2.appreciationScore as Rating2_appreciationScore,
  Rating2.userId as Rating2_userId,
  Song1.year as Song1_year,
  Song1.title as Song1_title,
  Song1.filePath as Song1_filePath,
  Song1.artistId as Song1_artistId,
  Song1.id as Song1_id
From
  Song Song1
left outer join Rating as Rating2 on (Song1.id = Rating2.songId)
]]>

</script>
