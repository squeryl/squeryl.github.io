---
layout: manual
title: Insert, Update and Delete
headtitle: Insert, Update and Delete - 
---

Insert
------

The insert mechanism is the least surprising of all :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

val herbyHancock =  
artists.insert(new Artist("Herby Hancock"))

val ponchoSanchez =  
artists.insert(new Artist("Poncho Sanchez"))

val theMeters =  
artists.insert(new Artist("The Meters"))  
]]>

</script>

Objects that extend KeyedEntity\[K\] where K is a numeric type  
will have their id field assigned their newly created primary key  
value (the mechanism for generating keys is specific to each
DatabaseAdaptor).

Update
------

There are two forms of updates :

1\. Full

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

watermelonMan.title = “The Watermelon Man”  
watermelonMan.year = watermelonMan.year + 1  
songs.update(watermelonMan)  
]]>

</script>

2\. Partial :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

update(songs)(s =>  
where(s.title === "Watermelon Man")  
set(s.title := "The Watermelon Man",  
s.year := s.year.~ + 1)  
)  
]]>

</script>

The SQL will be :

<script type="syntaxhighlighter" class="brush: sql">

<![CDATA[  
update Song set  
title = ?,  
year = (year + ?)  
Where  
(title = ?)  
]]>

</script>

A partial update must have a where clause, otherwise you’ll get a
compilation error.  
This is to prevent you from updating all rows in a table by accident.

To update all rows use the setAll function :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[  
update(songs)(s => setAll(s.year := s.year.~ + 1))  
]]>

</script>

Delete
------

Delete is done either by key (when objects extend KeyedEntity\[K\]),
or  
with a boolean clause via the table’s deleteWhere method,  
example of Table\[PlaylistElement\].deleteWhere usage :

<script type="syntaxhighlighter" class="brush: sql">

<![CDATA[

def removeSong(song: Song) =  
playlistElements.deleteWhere(ple => ple.songId === song.id)

def removeSongOfArtist(artist: Artist) =  
playlistElements.deleteWhere(ple =>  
(ple.playlistId === id) and  
(ple.songId in from(songsOf(artist.id))(s => select(s.id)))  
)

]]>

</script>

Batched updates and Inserts
---------------------------

org.squeryl.Table\[A\] has insert and update methods that take an
Iterable\[A\]. Invoking them does the update in a single roundtrip to
the database via JDBC’s batched update functionality.

The advantage of this is obviously making 1 trip to the DB versus N
trips (given an iterable with N elements).

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

addresses.insert(List(  
new Address("St-Dominique",14, None,None,None),  
new Address("St-Urbain",23, None,None,None),  
new Address("Sherbrooke",1123, None,Some(454),Some("B"))  
))

addresses.insert(List(  
new Address("Van Horne",14, None,None,None)  
))

val q = addresses.where(a => a.streetName in streetNames)

assertEquals(4, q.Count : Long, "batched update test failed")

// The update here is one in a single DB trip :

addresses.update(q.map(a =>{a.streetName += "Z"; a}))

val updatedStreetNames = List("Van HorneZ", "SherbrookeZ", "St-UrbainZ",
"St-DominiqueZ")

val updatedQ = addresses.where(a => a.streetName in updatedStreetNames)

]]>

</script>

Active Record pattern
---------------------

Another option available for inserting objects is through the use of
the  
convenience save method. This method allows the entity to be persisted  
without the invocation of the insert method of the respective table :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

import MusicDb._

new Artist("Herby Hancock").save  
]]>

</script>

In the same manner, Updates can also be done with the usage of the  
Active Record pattern :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

import MusicDb._

watermelonMan.title = "The Watermelon Man"  
watermelonMan.year = watermelonMan.year + 1  
watermelonMan.update  
]]>

</script>

It’s important to note that the contents of the Schema object were
imported  
before the usage of those convenience methods.

While this makes your code look smaller and less cluttered, it may not
work  
correctly when dealing with some specific corner cases, like a class
that has a  
common structure and is mapped to two or more different tables.
