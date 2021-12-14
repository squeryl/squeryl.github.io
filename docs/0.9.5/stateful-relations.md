---
layout: manual
title: Stateful Relations
headtitle: Stateful Relations - 
---

### Declaring Stateful Relations

Stateful relations have the same methods as stateless ones, the main
difference is that wherever you have a Query\[A\] in a stateless  
becomes an Iterable\[A\] in a stateful relation.

Stateful relations are defined exactly the same way as their stateless
counterpart, with the exception that the **leftStateful** and
**rightStateful** methods are  
invoke instead of **left** and **right** :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
class Course(val subjectId: Long) extends SchoolDb2Object {

  //students is a ManyToMany[Student,CourseSubscription], it extends Query[Students]
  lazy val students = SchoolDb2.courseSubscriptions.leftStateful(this)
}

class Student(val firstName: String, val lastName: String) extends SchoolDb2Object {
  //courses is a ManyToMany[Course,CourseSubscription], it extends Query[Course]
  lazy val courses = SchoolDb2.courseSubscriptions.rightStateful(this)
}

class Course(val subjectId: Long) extends SchoolDb2Object {
  lazy val subject: ManyToOne[Subject] = SchoolDb.subjectToCourses.rightStateful(this)
}

class Subject(val name: String) extends SchoolDb2Object {
  lazy val courses: OneToMany[Course] = SchoolDb.subjectToCourses.leftStateful(this)
}
]]>

</script>

### StatefulOneToMany and StatefulManyToOne

Methods in stateful relations are very similar than their stateless
counterpart, their signatures are slightly different, and have a refresh
method. Note the **relation** property, it is the stateless relation
objects that provides the persistence related functionality. A stateful
relation is in fact a wrapper over a stateless one, with caching.

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
class StatefulOneToMany[M](val relation: OneToMany[M]) extends Iterable[M] {

  def refresh: Unit
  def associate(m: M):Unit
  def deleteAll: Int
}

class StatefulManyToOne[O <: KeyedEntity[_]](val relation: ManyToOne[O]) {

  def refresh: Unit
  def one: Option[O]
  def assign(o: O): Unit
  def delete: Boolean
}
]]>

</script>

### StatefulManyToMany

The similarity between StatefulManyToMany and ManyToMany is also very
apparent :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
class StatefulManyToMany[O <: KeyedEntity*],A <: KeyedEntity[*](val relation: ManyToMany[O,A]) extends Iterable[O] {

  def refresh: Unit
  def associate(o: O, a: A)
  def associate(o: O): A
  def dissociate(o: O): Boolean
  def dissociateAll: Int
  def associations: Iterable[A]
}
]]>

</script>
