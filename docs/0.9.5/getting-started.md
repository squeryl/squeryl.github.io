---
layout: manual
title: Getting Started
headtitle: Getting Started - 
---

### The recommended version is 0.9.5-7

Supported Scala versions are 2.11.x, 2.10.x, 2.9.3, 2.9.2, 2.9.1, 2.9.0

SBT
---

After having setup your project to use SBT (see [how to setup
SBT](http://github.com/harrah/xsbt/wiki))  
Declare the Squeryl dependency in the SBT project definiition

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

libraryDependencies ++= Seq(
  “org.squeryl” “squeryl” % “0.9.5-7”,
  yourDatabaseDependency
)

//yourDatabaseDependency is one of the supported databases:

val h2 = “com.h2database” % “h2” % “1.2.127”
val mysqlDriver = “mysql” % “mysql-connector-java” % “5.1.10”
val posgresDriver = “postgresql” % “postgresql” % “8.4-701.jdbc4”
val msSqlDriver = “net.sourceforge.jtds” % “jtds” % “1.2.4”
val derbyDriver = “org.apache.derby” % “derby” % “10.7.1.1”
]]>

</script>

***Note*** : *The Oracle driver is not available via SBT/Maven, you will have to downloaded manually.*

The remaining steps are:

1.  [Bootstrap the session
    factory](http://squeryl.org/sessions-and-tx.html)
2.  [Define, and initialise a
    schema](http://squeryl.org/schema-definition.html)

You should now be ready to insert, update, delete data, and of course
run queries.

### Manual dependency management

If you want to manage your dependencies without SBT or Maven, you will
need the following jars:

-   The Scala runtime jar, version 2.11.x, 2.10.x, 2.9.3, 2.9.2, 2.9.1, 2.9.0
-   The Squeryl jar found
    [here](http://github.com/max-l/Squeryl/downloads)
-   The CGLIB cglib-nodep-2.2.jar, found
    [here](http://sourceforge.net/projects/cglib/files/)
-   A JDBC driver for your database, see [supported
    databases](./supported-databases.html)
