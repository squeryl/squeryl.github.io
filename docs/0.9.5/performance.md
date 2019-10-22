---
layout: manual
title: Performance Tuning
headtitle: Performance Tuning - 
---

Caching : Your Mileage *will* vary
----------------------------------

Benefits from caching is, quite evidently, highly dependent on the data
access patterns of an application.

### 

Reducing Squeryl overhead
-------------------------

Most of Squeryl’s overhead over raw JDBC comes from AST construction.

Query\[A\] are immutable, and can be shared.
--------------------------------------------

Query AST construction is unlikely to be the first bottleneck in an
application, but it can become one,  
the good news is that

### use MutableQuery\[A\] when
