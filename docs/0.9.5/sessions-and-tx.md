---
layout: manual
title: Sessions and Transactions
headtitle: Sessions and Transactions - 
---

Bootstrap with org.squeryl.SessionFactory
-----------------------------------------

**SessionFactory.concreteFactory** must be initialized before Squeryl
transactions can be invoked :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
import org.squeryl.SessionFactory

Class.forName(“org.postgresql.Driver”);

SessionFactory.concreteFactory = Some(()=>
  Session.create(
    java.sql.DriverManager.getConnection(“…”),
    new PostgreSqlAdapter))
]]>

</script>

After the initialization of SessionFactory.concreteFactory, the
**transaction** and **inTransaction** block functions (call by names)
become available :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
import org.squeryl.PrimitiveTypeMode._

//Squeryl database interaction must occur in a transaction block:
transaction {
  books.insert(new Author(1, “Michel”,“Folco”))
  val a = from(authors)(a=> where(a.lastName === “Folco”) select(a))
}

inTransaction {
  authors.where(a=> a.lastName === “Pouchkine”)

  //when in a transaction, the current session can be obtained with:
  val s = Session.currentSession
}
]]>

</script>

The ‘transaction’ function binds the session to the current thread for the duration  
of the block, so any method called directly **and indirectly** from the block  
will be *in the context* of the transaction.

### Distinction between transaction and inTransaction

-   **’transaction’** causes a new transaction to begin and commit after
    the block’s execution, or rollback if an exception occurs. Invoking
    a transaction always cause a new one to be created, even if called
    in the context of an existing transaction.

<!-- -->

-   **’inTransaction’** will create a new transaction if none is in
    progress and commit it upon completion or rollback on exceptions. If
    a transaction already exists, it has no effect, the block will
    execute in the context of the existing transaction. The
    commit/rollback is handled in this case by the parent transaction
    block.

### Using Squeryl under external transaction management

When used within a framework that already manages transaction, for
example a web framework, Squeryl must use the java.sql.Connection that
belongs to the “current” context (usually implemented via thread local
storage ) this can be accomplished by initializing
SessionFactorye.externalTransactionManagementAdapter in the bootstrap
code (before any Squeryl code gets to execute).

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
SessionFactorye.externalTransactionManagementAdapter = Some(
  () => new Session(
    …obtain the current session here …
    new OracleAdapter))
]]>

</script>

### Managing transactions manually

Given an org.squeryl.Session, Squeryl statements can be issued with the
**using** function :

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[
using(squerylSession) {
  // your Squeryl code here…
}
]]>

</script>

-   **Note1** : with **using** the user code has the responsibility to
    close the connection.
-   **Note2** : calling **inTransaction {}** within a **using {}** block
    has no effect, i.e. the inTransaction behaves as if there was a
    transaction, i.e. it does nothing (*see inTransaction*).
