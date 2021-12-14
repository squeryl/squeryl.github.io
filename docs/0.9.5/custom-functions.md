---
layout: manual
title: Custom Functions
headtitle: Custom Functions - 
---

Suppose your database has a SHA1 function that takes a String and
returns a String

You can define :

<script type="syntaxhighlighter" class="brush: scala">
<![CDATA[

class SHA1(e: StringExpression[String], m:OutMapper[String]) 
  extends FunctionNode[String](“sha1”, Some(m), Seq(e)) 
  with StringExpression[String]

def sha1(e: StringExpression[String])(implicit m:OutMapper[String]) = new SHA1(e,m)
]]>
</script>

Then you can use the sha1 function in expressions :

<script type="syntaxhighlighter" class="brush: scala">
<![CDATA[

def sha1Of(id: Long) =
  from(aTable)(u=> where(u.id === id) select(&(sha1(u.aStringField)))

def isSha1Equal(id: Long, s: String) =
  from(aTable)(u=> where(u.id = id and sha1(u.aStringField) = s)).Count == 1
]]>
</script>

The arguments and return type of your custom function need be one of the
following Squeryl types (defined in org.squeryl.dsl) :

<script type="syntaxhighlighter" class="brush: scala">
<![CDATA[
trait NumericalExpression[A]
trait StringExpression[B]
trait DateExpression[C]
trait BooleanExpression[D]
]]>
</script>

Where: 

-   A is Int, Long, Float, Double, BigDecimal or Option\[Int\],
    Option\[Long\], etc…
-   B is String or Option\[String\]
-   C is Date, Timestamp, Option\[Date\] or Option\[Timestamp\]
-   D is Boolean or Option\[Boolean\]

**Note**: you need 0.9.5-Beta or higher to have the appropriate implicit
OutMapper\[A\] in scope (the implicit def is defined in an ancestor of
PrimitiveTypeMode, CustomTypeMode and RecordTypeMode
(org.squeryl.dsl.TypeArithmetic))

<script type="syntaxhighlighter" class="brush: scala">
<![CDATA[

// For Squeryl versions before 0.9.5-Beta, OutMappers need to be
// created and given as argument to FunctionNode

def m1:OutMapper[String] = new OutMapper[String] {
  def doMap(rs: ResultSet) = rs.getString(index)
  def sample = “”
}

class SHA1(e: StringExpression[String])
  extends FunctionNode[String](“sha1”, Some(m1), Seq(e)) with
  StringExpression[String]

def sha1(e: StringExpression[String]) = new SHA1(e,m)

def m2:OutMapper[Option[StringType]] = new OutMapper[Option[String]] {
  def doMap(rs: ResultSet) = {
    val v = rs.getString(index)
    if(rs.wasNull)
      None
    else
      Some(v)
  }
  def sample = Some(“”)
}
]]>

</script>
