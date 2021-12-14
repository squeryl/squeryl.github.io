---
layout: manual
title: Relations
headtitle: Relations - 
---

### Stateful vs Stateless relations

Relations allow foreign key relationships to appear as collection like
objects. There are two types of relations in Squeryl : OneToMany and
ManyToMany, and each has a stateful counterpart.

Stateful relations owe their name to the fact that they maintain members
of the relation in memory. A stateful relation loads all of it’s members
in memory when it is accessed for the first time, and they are kept with
the relation. When an object is inserted or updated, the modified (or
inserted) object is persisted in the database and kept in memory
‘inside’ the relation object.

From here on, stateless relations will be referred simply as relations,
stateful relations are described [here](stateful-relations.html) .

### OneToMany

<script type="syntaxhighlighter" class="brush: scala">


<![CDATA[

object SchoolDb extends Schema {

  val courses = table[Course]
  val subjects = table[Subject]

  val subjectToCourses =
    oneToManyRelation(subjects, courses).
    via((s,c) => s.id === c.subjectId)
}

class Course(val subjectId: Long) extends SchoolDb2Object {
  lazy val subject: ManyToOne[Subject] = SchoolDb.subjectToCourses.right(this)
}

class Subject(val name: String) extends SchoolDb2Object {
  lazy val courses: OneToMany[Course] = SchoolDb.subjectToCourses.left(this)
}
]]>

</script>

The traits OneToMany and ManyToOne both extends Query\[\], they have an
**assign** method  
that assigns the keys while leaving the database unchanged. The
**associate** calls assign  
**and** persists the changes to the database.

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

trait OneToMany[M] extends Query[M] {

  /* @param the object on the ‘many side’ to be associated with this
   * 
   * Sets the foreign key of ‘m’ to refer to the primary key of the ‘one’ instance
   */
  def assign(m: M): M

  /* Calls ‘assign(m)’ and persists the changes the database, by inserting or updating ‘m’, depending
   * on if it has been persisted or not.
   */
  def associate(m: M): M

  def deleteAll: Int
}

trait ManyToOne[O <: KeyedEntity[_]] extends Query[O] {

  def assign(one: O): O

  def delete: Boolean
}
]]>

</script>

### ManyToMany

A ManyToMany relation has a table in the *middle* called the
**association table**, it has a foreign key for the *left* and *right*
side. It is a symmetrical relation, so the choice of left and right is
arbitrary.  
In the following example we have a course to student relation, in which
the left side is the course and the right side is the student.  
The association table can have other fields besides the *left* and
*right* foreign keys, in the example, **CourseSubscription** has a
**grade:Float** column.

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

class CourseSubscription(val courseId: Int, val studentId: Int, val grade: Float) extends KeyedEntity[CompositeKey2[Int,Int]] {
  def id = compositeKey(courseId, studentId)
}

object SchoolDb extends Schema {

  val students = table[Student]
  val courses = table[Course]

  // courseSubscriptions is a ManyToManyRelation, it extends Table[CourseSubscriptions]
  val courseSubscriptions =
    manyToManyRelation(courses, students).
    via[CourseSubscription]((c,s,cs) => (cs.studentId = s.id, c.id = cs.courseId))
}

class Course(val subjectId: Long) extends SchoolDb2Object {

  //students is a ManyToMany[Student,CourseSubscription], it extends Query[Students]
  lazy val students = SchoolDb2.courseSubscriptions.left(this)
}

class Student(val firstName: String, val lastName: String) extends SchoolDb2Object {

  //courses is a ManyToMany[Course,CourseSubscription], it extends Query[Course]
  lazy val courses = SchoolDb2.courseSubscriptions.right(this)
}
]]>

</script>

Examples of many to many relation usage :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

// the following two lines are equivalent, they both create  
// and insert a CourseSubscription, with the proper foreign keys :

physics.students.associate(olga)

olga.courses.associate(physics)

//physics.students is a Query[Student] selecting all stutents  
//in the physics course :

println(“students of physics :”)
for(s <- physics.students)
  println(s.fullName)

// we can get acces the association objects of  
// a relation member, here we have the CourseSubscription  
// of the physics course:
physics.students.associations: Query[CourseSubscription]

// just like for OneToMany, we have the ‘assign’ methods,  
// that only creates the association object without inserting :  
val cs:CourseSubscription = olga.courses.assign(physics)

// cs must be manually inserted for the association to be
// persistent:
SchoolDb.courseSubscriptions.insert(cs)
]]>

</script>

#### The trait ManyToMany\[O,A\]

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
/* This trait is what is referred by both the left and right side of a manyToMany relation.
 * Type parameters are:
 * O: the type at the “other” side of the relation
 * A: the association type i.e. the entity in the “middle” of the relation
 *
 * Object mapping to the “middle” entity are called “association objects”
 *
 * this trait extends Query\[O\] and can be queried against like a normal query.
 *
 * Note that this trait is used on both “left” and “right” sides of the relation,
 * but in a given relation
 */
trait ManyToMany[O <: KeyedEntity*],A <: KeyedEntity[*] extends Query[O] {

  /* @param a: the association object
   *
   * Sets the foreign keys of the association object to the primary keys of the left and right side,
   * this method does not update the database, changes to the association object must be done for
   * the operation to be persisted. Alternatively the method ‘associate(o, a)’ will call this assign(o, a)
   * and persist the changes.
   */
  def assign(o: O, a: A): A

  /* @param a: the association object
   *
   * Calls assign(o,a) and persists the changes the database, by inserting or updating ‘a’, depending
   * on if it has been persisted or not.
   */
  def associate(o: O, a: A): A

  /* Creates a new association object ‘a’ and calls assign(o,a)
   */
  def assign(o: O): A

  /* Creates a new association object ‘a’ and calls associate(o,a)
   *
   * Note that this method will fail if the association object has NOT NULL constraint fields appart from the
   * foreign keys in the relations
   *
   */
  def associate(o: O): A

  /* Causes the deletion of the ‘Association object’ between this side and the other side
   * of the relation.
   * @return true if ‘o’ was associated (if an association object existed between ‘this’ and ‘o’) false otherwise
   */
  def dissociate(o: O): Boolean

  /* Deletes all “associations” relating this “side” to the other
   */
  def dissociateAll: Int

  /* a Query returning all of this member’s association entries
   */
  def associations: Query[A]

  /* @return a Query of Tuple2 containing all objects on the ‘other side’ along with their association object
   */
  def associationMap: Query[(O,A)]
}
]]>

</script>

### Defining foreign key constraints with relations

The org.squeryl.Schema.create method will create relations based on the
declarations in the schema as the next example shows, note the different
signatures of **constrainReference**. The applyDefaultForeignKeyPolicy
method will determine how the relations are declared at the schema
level, each relation can then redefine the default.  
Calling unConstrainReference in applyDefaultForeignKeyPolicy will cause
foreign key constraints to be inhibited.

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

object SchoolDb2 extends Schema {

  val professors = table[Professor]
  val students = table[Student]
  val courses = table[Course]
  val subjects = table[Subject]

  val courseAssignments =
    manyToManyRelation(professors, courses).
     via[CourseAssignment]((p,c,a) => (p.id = a.professorId, a.courseId = c.id))

  val courseSubscriptions =
    manyToManyRelation(courses, students).
      via[CourseSubscription]((c,s,cs) => (cs.studentId = s.id, c.id = cs.courseId))

  val subjectToCourses =
    oneToManyRelation(subjects, courses).
      via((s,c) => s.id === c.subjectId)

  // the default constraint for all foreign keys in this schema:
  override def applyDefaultForeignKeyPolicy(foreignKeyDeclaration: ForeignKeyDeclaration) =
    foreignKeyDeclaration.constrainReference

  //now we will redefine some of the foreign key constraints:
  //if we delete a subject, we want all courses to be deleted
  subjectToCourses.foreignKeyDeclaration.constrainReference(onDelete cascade)

  //when a course is deleted, all of the subscriptions will get deleted:
  courseSubscriptions.leftForeignKeyDeclaration.constrainReference(onDelete cascade)
}
]]>

</script>

### Performance considerations

A significant part of optimizing a database abstraction layer is to
choose for every situation the  
right balance between fine and large grained retrieval, and the optimal
mix of laziness and eagerness.

Relations are lazy, and are subject to the [N+1
problem](http://stackoverflow.com/questions/97197/what-is-the-n1-selects-problem)
. If we have a *deep* relation chain, it is often preferable to use a
query, for example if we have an N level *relation chain* : R1 x R2,…,
Rn, looping through relations will cause :

1\* \|R2\| \* \|R3\| \* …. \* \|Rn-1\| database trips

where \|Rk\| is the number of rows in the Rk part of the relation

In such a case, a single query is clearly preferable since it will fetch
all objects in a single database round trip.
