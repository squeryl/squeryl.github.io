---
layout: manual
title: Composite Keys
headtitle: Composite Keys -
---

### Declaring composite keys for self documentation and fulfilling KeyedEntity’s contract

The most compelling reason to use composite key is to implement the
KeyedEntity trait (and enable it’s functionality). It also has the
advantage of documenting the fact that fields/columns form a composite
key. Another benefit is that equality expressions can be shortened.

Consider the following example where CourseAssignment is a
KeyedEntity\[CompositeKey2\[Long,Long\]\]

<script type="syntaxhighlighter" class="brush: scala">
<![CDATA[
class CourseAssignment(val courseId: Long, val professorId: Long) extends KeyedEntity[CompositeKey2[Long,Long]] {
  def id = compositeKey(courseId, professorId)
}
]]>
</script>

CourseAssignment.id is a unique identifier for a CourseAssignment. In
this case **id** is what makes the class a
KeyedEntity\[CompositeKey2\[Long,Long\]\]. A class can have any number
of composite keys, a composite key is not necessarily a primary key.

-   **Important**: Squeryl will create a uniqueness constraint on
    composite keys that form the **id** of a KeyedEntity\[\_\]

<!-- -->

-   **Important**: a compositeKey cannot be a **val** it must be a
    **def**

A composite key can be used in any equality expression (x === y),
against a  
composite key or a tuple that is compatible, i.e. that has the same type
parameters, ex.:

-   CompositeKey3\[String,Long,float\] is compatible with
    Tuple3(String,Long,Float)

<!-- -->

-   CompositeKey2\[Long,Int\] is **not** compatible with
    Tuple2(Long,Long)

**Note** CompositeKeys cannot be used as binding expressions in
relations (see : [issue
25](http://www.assembla.com/spaces/squeryl/tickets/25))

<script type="syntaxhighlighter" class="brush: scala">
<![CDATA[
val aCourseAssignment = courseAssignments.where(…).single

val q1 = courseAssignments.where(_.id === aCourseAssignment.id)

val q2 = courseAssignments.where(_.id ===(113243L, 26543546L))

println(q.statement)
]]>
</script>

<script type="syntaxhighlighter" class="brush: sql">
<![CDATA[
Select
CourseAssignment1.professorId as CourseAssignment1_professorId,
CourseAssignment1.courseId as CourseAssignment1_courseId
From
CourseAssignment CourseAssignment1
Where
((CourseAssignment1.courseId = 113243) and
(CourseAssignment1.professorId = 26543546))
]]>
</script>
<script type="syntaxhighlighter" class="brush: scala">
<![CDATA[
val aCourseAssignment = courseAssignments.where(…).single

val q1 =
  courseAssignments.where(
    _.id.courseId = aCourseAssignment.courseId and
      _.id.professorId = aCourseAssignment.professorId)
]]>
</script>
