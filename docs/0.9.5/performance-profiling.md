---
layout: manual
title: Performance profiling
---

Performance statistics can be collected,

This [example profile](/profileOfH2Tests.html) has been collected during
the an execution of the test suite.

### Profiling load on a single VM

<script type="syntaxhighlighter" class="brush: scala">

<![CDATA[

// This will create an H2 database file named ‘db-usage-stats.h2.db’

val localH2SinkStatisticsListener = LocalH2SinkStatisticsListener.initializeOverwrite(“\~/db-usage-stats”)

// Sessions that are to be profiled must be given an instance of the
// LocalH2SinkStatisticsListener, note the 3rd parameter of the Session constructor:

SessionFactory.concreteFactory = Some(()=>
  new Session(
    java.sql.DriverManager.getConnection(….),
    new H2Adapter,
    Some(localH2SinkStatisticsListener)
)

// … run your load or usage simulation here …

// Now the h2 database db-usage-stats.h2.db is filled with usage stats,
// the following method will generate a statistic summary charting the top 10 queries
// with longest run time, highest cumulated run time, etc:

localH2SinkStatisticsListener.generateStatSummary(new java.io.File(“./profileOfH2Tests.html”), 10)
]]>

</script>

### Profiling load on multiple VMs

The H2 database table are meant to be mergeable, the
org.squeryl.logging.UsageProfileConsolidator utility can be used to
merge the stats that have been collected in multiple VMs.

     java org.squeryl.logging.UsageProfileConsolidator <h2FileForConsolidatedStatsProfile> <list of h2 files to consolidate>
