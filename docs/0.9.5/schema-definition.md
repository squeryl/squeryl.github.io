---
layout: manual
title: Schema Definition
headtitle: Schema Definition - 
---

Defining a Schema
-----------------

Scala classes are mapped to tables via instances of
org.squeryl.Table\[T\],  
that are grouped in a org.squeryl.Schema singleton.

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
import org.squeryl.PrimitiveTypeMode._
import org.squeryl.Schema
import org.squeryl.annotations.Column
import java.util.Date
import java.sql.Timestamp

class Author(val id: Long,
  val firstName: String,
  val lastName: String,
  val email: Option[String]) {
    def this() = this(0,“”,“”,Some(“”))
  }

  // fields can be mutable or immutable
  class Book(val id: Long,
      var title: String,
      @Column(“AUTHOR_ID”) // the default ‘exact match’ policy can be overriden
      var authorId: Long,
      var coAuthorId: Option[Long]) {

  def this() = this(0,“”,0,Some(0L))
}

class Borrowal(val id: Long,
  val bookId: Long,
  val borrowerAccountId: Long,
  val scheduledToReturnOn: Date,
  val returnedOn: Option[Timestamp],
  val numberOfPhonecallsForNonReturn: Int)

object Library extends Schema {

  //When the table name doesn’t match the class name, it is specified here:  
  val authors = table[Author](“AUTHORS”)

  val books = table[Book]
  val borrowals = table[Borrowal]
}
]]>

</script>

### Columns attributes

Columns and group of columns can have attributes declared via the
on/declare syntax. Inside the example schema defined above, we could
influence the DDL generation with the following declarations :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

object Library extends Schema {
  …
  …
  on(borrowals)(b => declare(
    b.numberOfPhonecallsForNonReturn defaultsTo(0),
    b.borrowerAccountId is(indexed),
    columns(b.scheduledToReturnOn, b.borrowerAccountId) are(indexed)))

  on(authors)(s => declare(
    s.email is(unique,indexed(“idxEmailAddresses”)), //indexes can be named explicitely
    s.firstName is(indexed),
    s.lastName is(indexed, dbType(“varchar(255)”)), // the default column type can be overriden
    columns(s.firstName, s.lastName) are(indexed)))
}
]]>

</script>

Of course, not all combinations of column attributes make sense, the
valid combinations are :

|                     | non numeric column | numeric column | column group |
|---------------------|--------------------|----------------|--------------|
| **indexed**         | Y                  | Y              | Y            |
| **unique**          | Y                  | Y              | Y            |
| **autoIncremented** | N                  | Y              | N            |
| **defaultsTo**      | Y                  | Y              | N            |

-   "Note\*: the uniqueness of KeyedEntity\[\].id columns is not
    overridable, it always gets declared as ‘primaryKey’

### Schema generation (DDL)

Schema.create will connect to the database and create all tables,
constraints, indexes, etc. While this is usefull for development phases,
when a system has been in production long enough it is often more
convenient to generate the schema, and evolve it manually.

Use Schema.printDdl to print your schema :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
def printDdl: Unit = printDdl(println(_))

def printDdl(pw: PrintWriter): Unit = printDdl(pw.println(_))

def printDdl(pw: String => Unit): Unit = {…}
]]>

</script>

Mapping fields to Columns
-------------------------

Squeryl applies the principle of *Convention over Configuration* :  
The class to table and field to column correspondence is determined  
by name equivalence. It is possible to override a field’s column name  
with the org.squeryl.annotations.Column annotations and the class’s  
table name table with the
org.squeryl.Schema.table\[T\](tableName:String)  
method. as illustrated in the previous example.

The Column annotation can also be used to redefine default length  
for String/varchar columns (and also for other types although it  
should rarely be necessary).

### Nullable columns are mapped with Option\[\] fields

The default (and strongly recommended) way of mapping nullable columns  
to fields is with the Option\[\] type. If you use Squeryl to create  
(or generate) your schema, all fields have a not null constraint,  
and Option\[\] fields are nullable.

-   **Important** : If a class has an Option\[\] field, it becomes
    **mandatory**  
    to implement a zero argument constructor that initializes Option\[\]
    fields  
    with Some() instances (like the Book class in the example above).  
    Failing to do so will cause an exception to be thrown  
    when the table will be instantiated. The reason for this is that
    type erasures  
    imposed by the JVM prevents from reflecting on the Option\[\] type
    parameter.  
    This constraint could be relaxed in a future version by a compiler
    plugin  
    that would *tell* Squeryl the erased type information.

### Correspondance of field types to database column types

| Java/JDBC  | Oracle      | PostgreSql       | DB2        | MySql      | H2         | MS SQL     | Derby      |
|------------|-------------|------------------|------------|------------|------------|------------|------------|
| int        | number      | integer          | int        | int        | int        | int        | integer    |
| long       | number      | bigint           | bigint     | bigint     | bigint     | bigint     | bigint     |
| float      | float       | real             | real       | float      | real       | real       | real       |
| double     | real        | double precision | double     | double     | double     | float      | double     |
| BigDecimal | decimal     | numeric          | decimal    | decimal    | decimal    | decimal    | decimal    |
| String     | varchar2(x) | varchar(x)       | varchar(x) | varchar(x) | varchar(x) | varchar(x) | varchar(x) |
| Date       | date        | date             | date       | date       | date       | date       | date       |
| Timestamp  | date        | timestamp        | timestamp  | datetime   | timestamp  | datetime   | timestamp  |
| byte\[\]   | blob        | bytea            | blob       | blob       | binary     | varbinary  | blob(1M)   |
| boolean    | number(1)   | boolean          | char(1)    | boolean    | boolean    | bit        | char(1)    |
| UUID       | char(36)    | uuid             | char(36)   | char(36)   | uuid       | char(36)   | char(36)   |

### Enumerations

Enumerations are persisted by ‘int’ columns

-   Explicitely specifying the index values in Enumerations is
    **strongly recomended**

<!-- -->

-   Classes with an Enumeration field **require** a zero arg constructor
    that gives a default value to all enumeration fields

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
object Tempo extends Enumeration {
  type Tempo = Value
  val Largo = Value(1, “Largo”)
  val Allegro = Value(2, “Allegro”)
  val Presto = Value(3, “Presto”)
}

class Song(name: String, tempo: Tempo) {
  def this() = this(“”,Tempo.Largo)
}
]]>

</script>

Choosing between primitive or custom types
------------------------------------------

You will have to decide if your table objects will be mapped with
primitive (Int. Long, Date, String etc.) or custom types. It’s a
question of tradeoffs :

### Primitive types

The main motivations for using primitive types are for performance and
simplicity.  
If a query returns N rows of objects with M fields primitive types  
will cause the the garbage collector to handle of N objects, while
same  
query using custom types will cause the creation of N \* M objects.

To use primitive types, simply import org.squeryl.PrimitiveTypeMode.\_  
in the scope where database objects and queries are defined :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
import org.squeryl.PrimitiveTypeMode._

class Song(val id: Long,
  val title: String,
  val artistId: Long,
  val filePath: Option[String],
  val year: Int)

from(songs)(s => where(s.title like “funk”) select(s))
]]>

</script>

that’s all there is to it.

<a name='disambiguate'></a>

-   **important** : in PrimitiveTypes mode there can be ambiguities
    between numeric operators

When using org.squeryl.PrimitiveTypeMode, the compiler will treat an
expression like the  
one in the next example as a Boolean. The .\~ function is needed to tell
the compiler that the  
left side is a node of TypedExpressionNode\[Int\] which will cause the
whole expression to be a  
LogicalBoolean which is what the where clause takes :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
from(songs)(s =>
  where(s.year.~ > 1965)
select(s))

// or use ‘gt’ instead of >:
from(songs)(s =>
  where(s.year gt 1965)
  select(s))
]]>

</script>

It is also needed in the following case :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
from(songs)(s => 
  where(s.year.~ + 10 = 1965)
  select(s))
// or use 'plus'
from(songs)(s =>
  where(s.year plus 10 = 1965)
  select(s)) 
]]>

</script>

This is required when using PrimitiveType mode. With custom types  
there is no ambiguity, since custom types are not (AnyVal) numerics.

If you are using primitive types, you should use the following operators
: div, times, plus, minus instead of /. \*, +, -

### Custom types

One motivation for using custom wrapper types is to allow fields  
to carry meta data along with validation, as in the next example.

Custom field types must inherit one of the subtypes of CustomType in the
package  
org.squeryl.customtypes, and import the
org.squeryl.customtypes.CustomTypesMode.\_  
into the scope where statements are defined.

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
import org.squeryl.customtypes.CustomTypesMode._
import org.squeryl.customtypes._

/* An example of trait that can be mixed into CustomType,
 * to add meta data and validation
 */
trait Domain[A] {
  self: CustomType =>

  def label: String
  def validate(a: A): Unit
  def value: A

  validate(value)
}

class Age(v: Int) extends IntField(v) with Domain[Int] {
  def validate(a: Int) = assert(a > 0, “age must be positive, got ” + a)
  def label = “age”
}

class FirstName(v: String) extends StringField(v) with Domain[String] {
  def validate(s: String) = assert(s.length \<= 50, “first name is waaaay to long : ” + s)
  def label = “first name”
}

class WeightInKilograms(v: Double) extends DoubleField(v) with Domain[Double] {
  def validate(d:Double) = assert(d > 0, “weight must be positive, got ” + d)
  def label = “weight (in kilograms)”
}

class Patient(val firstName: FirstName, val age: Age, val weight: WeightInKilograms)

val heavyWeights = from(patients)(p => where(p.weight > 250)
  select(p))
]]>

</script>
