---
layout: default
title: examples
headtitle: Examples -
---

<script type="syntaxhighlighter" class="brush: scala">
<![CDATA[
package org.squeryl.demos;

import org.squeryl.PrimitiveTypeMode._
import org.squeryl.adapters.H2Adapter
import org.squeryl.{Session, KeyedEntity, Schema}

// The root object of the schema. Inheriting KeyedEntity[T] is not mandatory
// it just makes primary key methods available (delete and lookup) on tables.
class MusicDbObject extends KeyedEntity[Long] {
  var id: Long = 0
}

class Artist(var name:String) extends MusicDbObject {

  // this returns a Query[Song] which is also an Iterable[Song]:
  def songs = from(MusicDb.songs)(s => where(s.artistId === id) select(s))

  def newSong(title: String, filePath: Option[String]) =
    MusicDb.songs.insert(new Song(title, id, filePath))
}

// Option[] members are mapped to nullable database columns,
// otherwise they have a NOT NULL constraint.
class Song(var title: String, var artistId: Long, var filePath: Option[String]) extends MusicDbObject {

  // IMPORTANT: currently classes with Option[] members **must** provide a zero arg
  // constructor where every Option[T] member gets initialized with Some(t:T).
  // or else Squeryl will not be able to reflect the type of the field, and an exception will
  // be thrown at table instantiation time.
  def this() = this(“”, 0, Some(“”))

  // the schema can be imported in the scope, to lighten the syntax:
  import MusicDb._

  // An alternative (shorter) syntax for single table queries:
  def artist = artists.where(a => a.id === artistId).single

  // Another alternative for lookup by primary key, since Artist is a
  // KeyedEntity[Long], it’s table has a lookup[Long](k: Long)
  // method available:
  def lookupArtist = artists.lookup(artistId)
}

class Playlist(var name: String, var path: String) extends MusicDbObject {

  import MusicDb._

  // a two table join:
  def songsInPlaylistOrder = from(playlistElements, songs)((ple, s) =>
    where(ple.playlistId = id and ple.songId = s.id) 
    select(s) 
    orderBy(ple.songNumber asc))

  def addSong(s: Song) = {

    // Note how this query can be implicitly converted to an Int since it returns
    // at most one row, this applies to all single column aggregate queries with no groupBy clause.
    // The nvl function in this example changed the return type to Int, from
    // Option[Int], since the ‘max’ function (like all aggregates, ‘count’ being the only exception).
    val nextSongNumber: Int =
      from(playlistElements)(ple =>
        where(ple.playlistId === id)
        compute(nvl(max(ple.songNumber), 0)))

    playlistElements.insert(new PlaylistElement(nextSongNumber, id, s.id))
  }

  // New concept: a group query with aggregate functions return GroupWithMeasures[K,M]
  // where K and M are tuples whose members correspond to the group by list and compute list
  // respectively.
  private def _songCountByArtistId =
    from(artists, songs)((a,s) =>
      where(a.id === s.artistId)
      groupBy(a.id)
      compute(count))

  // Queries are nestable just as they would in SQL
  def songCountForAllArtists =
    from(_songCountByArtistId, artists)((sca,a) =>
      where(sca.key === a.id)
      select((a, sca.measures)))

  // Unlike SQL, a function that returns a query can be nested
  // as if it were a query, notice the nesting of ‘songsOf’
  // allowing DRY persistence layers as reuse is enhanced.
  def latestSongFrom(artistId: Long) =
    from(songsOf(artistId))(s =>
      select(s)
      orderBy(s.id desc)
    ).headOption

  def songsOf(artistId: Long) =
    from(playlistElements, songs)((ple,s) =>
      where(id = ple.playlistId and ple.songId = s.id and s.artistId === artistId)
      select(s))
}

class PlaylistElement(var songNumber: Int, var playlistId: Long, var songId: Long)

object MusicDb extends Schema {
  val songs = table[Song]
  val artists = table[Artist]
  val playlists = table[Playlist]
  val playlistElements = table[PlaylistElement]
}

object KickTheTires {

  import MusicDb._

  //A Squeryl session is a thin wrapper over a JDBC connection:
  Class.forName(“org.h2.Driver”);
  val session = Session.create(
    java.sql.DriverManager.getConnection(“jdbc:h2:\~/test”, “sa”, “”),
    //Currently there are adapters for Oracle, Postgres, MySQL and H2:
    new H2Adapter
  )

  try {
    session.work {
      // database access code goes here
      test
      session.connection.commit
    }
  } catch {
    case e:Exception => session.connection.rollback
  }

  def test = {
    val herbyHancock = artists.insert(new Artist(“Herby Hancock”))
    val ponchoSanchez = artists.insert(new Artist(“Poncho Sanchez”))
    val mongoSantaMaria = artists.insert(new Artist(“Mongo Santa Maria”))

    val watermelonMan = herbyHancock.newSong(“Watermelon Man”, None)
    val besameMama = mongoSantaMaria.newSong(“Besame Mama”, Some(“c:/MyMusic/besameMama.flac”))
    val freedomSound = ponchoSanchez.newSong(“Freedom Sound”, None)
  }
}
]]>
</script>
