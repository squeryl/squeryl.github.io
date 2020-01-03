---
layout: manual
title: Select
headtitle: Select - 
---

The variations on Select
------------------------

Select statements return an immutable Query\[T\] that  
is itself a Queryable\[T\] and a lazy Iterable\[T\].

<script type="syntaxhighlighter" class="brush: scala">



class Artist(val id: Long, val name:String) {

def songs =  
from(MusicDb.songs)(s =\> where(s.artistId === id) select(s))

}  


</script>

The laziness here means that the query is sent to the database only  
when starting iteration, or in other words, when Iterable.iterator is
called.

The select function takes any legal Scala expression whose  
type determines the generic parameter of the Query\[R\]

<script type="syntaxhighlighter" class="brush: scala">


def select\[R\](r: =\>R): R  


</script>

The select expression will be evaluated for every row returned by the
query.

It is also possible to select with alternative (shorter but less
generic) syntax :

<script type="syntaxhighlighter" class="brush: scala">


class Song(var title: String, var artistId: Long) extends KeyedEntity {

import MusicDb.\_ // the schema can be imported in the scope

// A shorter syntax for single table queries :  
def artist = artists.where(a =\> a.id === artistId).single

// lookup by key is available because Artist extends  
// KeyedEntity\[Long\] :  
def lookupArtist = artists.lookup(artistId)  
}  


</script>

Note that the .lookup\[K\](k: K) i.e. lookup by key method on a
Table\[T\] is only  
available for Table\[T\] that are of the form :
Table\[KeyedEntity\[K\]\]

The classes Artist and Song in this example are part of a one to many
relation  
that can be accessed via the methods.

Nesting Sub Queries
-------------------

For the next examples, the following query will be nested as an inner
query  
into other queries :

<script type="syntaxhighlighter" class="brush: scala">


def songsInPlaylistOrder =  
from(playlistElements, songs)((ple, s) =\>  
where(ple.playlistId = id and ple.songId = s.id)  
select(s)  
orderBy(ple.songNumber asc)  
)  


</script>

### Sub Queries in the From clause :

The from clause takes Queryable\[T\]’s, a trait of Table\[T\] View\[T\]
**and** Query\[T\].  
Notice the Query\[Song\] **funkAndLatinJazz.songsInPlaylistOrder** in
the from clause :

<script type="syntaxhighlighter" class="brush: scala">



val songsFromThe60sInFunkAndLatinJazzPlaylist2 =  
from(funkAndLatinJazz.songsInPlaylistOrder)(s=\>  
where(s.id === 123)  
select(s)  
)  


</script>

### Sub Queries in the Where clause :

Joins can also be nested in the where clause just like in SQL :

<script type="syntaxhighlighter" class="brush: scala">


val songsFromThe60sInFunkAndLatinJazzPlaylist =  
from(songs)(s=\>  
where(s.id in  
from(funkAndLatinJazz.songsInPlaylistOrder)  
(s2 =\> select(s2.id))  
)  
select(s)  
)

for(s \<- songsFromThe60sInFunkAndLatinJazzPlaylist)  
println(s.title + " : " + s.year)  


</script>

The SQL generated for the above statement is :

<script type="syntaxhighlighter" class="brush: sql">


Select  
Song1.year as Song1\_year,  
Song1.title as Song1\_title,  
Song1.filePath as Song1\_filePath,  
Song1.artistId as Song1\_artistId,  
Song1.id as Song1\_id  
From  
Song Song1  
Where  
(Song1.id in  
(Select  
q3.Song5\_id as q3\_Song5\_id  
From  
(Select  
Song5.year as Song5\_year,  
Song5.title as Song5\_title,  
Song5.filePath as Song5\_filePath,  
Song5.artistId as Song5\_artistId,  
Song5.id as Song5\_id  
From  
PlaylistElement PlaylistElement4,  
Song Song5  
Where  
((PlaylistElement4.playlistId = ?) and (PlaylistElement4.songId =
Song5.id))  
Order By  
PlaylistElement4.songNumber Asc  
) q3  
))



</script>

Doing a 3 level nested join is by no means necessary and serves  
no other purposes than demonstration.

In addition to the in() operator, exists() and notExists() can be used
in a  
where clause. They correspond to EXISTS and NOT EXISTS in SQL. Any
query  
nested with in(), exists(), or notExists() can also refer to queries in
an  
outer scope.

For example:

<script type="syntaxhighlighter" class="brush: scala">


val studentsWithAnAddress =  
from(students)(s =\>  
where(exists(from(addresses)((a) =\> where(s.addressId === a.id)
select(a.id))))  
select(s)  
)  


</script>

The SQL generated for the above statement is :

<script type="syntaxhighlighter" class="brush: sql">


Select  
Student8.name as Student8\_name,  
Student8.age as Student8\_age,  
Student8.isMultilingual as Student8\_isMultilingual,  
Student8.lastName as Student8\_lastName,  
Student8.id as Student8\_id,  
Student8.addressId as Student8\_addressId,  
Student8.gender as Student8\_gender  
From  
Student Student8  
Where  
(exists(Select  
Address11.id as Address11\_id  
From  
Address Address11  
Where  
(Student8.addressId = Address11.id)  
) )  


</script>

Select Distinct
---------------

Calling the **.distinct** method on a Query\[\] creates a copy of it
that has a ‘distinct’ select clause :

<script type="syntaxhighlighter" class="brush: scala">


from(songs)(s =\> select(&(s.title))).distinct  


</script>

For Update
----------

Calling the **.forUpdate** method on a Query\[\] creates a copy of it
that has a ‘forUpdate’ locking directive :

<script type="syntaxhighlighter" class="brush: scala">


aTable.where(t =\> t.aField === aValue).forUpdate  


</script>
