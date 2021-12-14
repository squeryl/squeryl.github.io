---
layout: default
title: Submiting bugs and Contributing
headtitle: Contributing - 
---

Help us make Squeryl a world class game changing technology
-----------------------------------------------------------

There are no shortage of interesting problems requiring elegant
solutions. Brilliant and talented minds are a welcome addition to
Squeryl’s DNA.

A snippet of code to reproduce the bug is ideal.
------------------------------------------------

-   The bug will get solved quicker

<!-- -->

-   It will help to grow the test suite and make  
    it harder for Squeryl to regress !

When that is not possible, sending the AST of the query  
(with Query\[T\].dumpAst) is also helpful :

<script type="syntaxhighlighter" class="brush: scala">
<![CDATA[
val q =
  from(artists, songs)((a,s) =>
    where(a.id === s.artistId)
    groupBy(a.id)
    compute(count))

println(q.dumpAst)
]]>
</script>

Submitting Ideas and Issues
---------------------------

The best place for submitting ideas is on the [discussion
group](http://groups.google.com/group/squeryl)

For issues and bugs we will start by using [GitHub’s issue
tracker](http://github.com/max-l/Squeryl/issues)

Contributing
------------

Like they say on GitHub : Fork me !!!

<http://github.com/max-l/Squeryl>

Patches are gladly accepted from their original author.  
Along with any patches, please state that the patch is your original
work and that you license  
the work to the Squeryl project under the project’s open source license.

Building from Sources
---------------------

Info on building is found
[here](http://github.com/max-l/Squeryl/blob/master/readme.txt)

At the moment the test mechanism is to call test code from the
org.squeryl.tests.allTestsOnH2  
this runs the tests within [H2](http://www.h2database.com/) as the name
implies.  
(Agreed… Squeryl should migrate to a real test framework, it will
eventually happen !!!)

Testing on H2 is really great because it doesn’t require you to setup a
database, if you build with  
[SBT](https://github.com/sbt/sbt) you will have an
[H2](http://www.h2database.com/) database ready to run the tests, and
if  
the test succeeds on H2, making it work on other DBs should be straight
forward.
