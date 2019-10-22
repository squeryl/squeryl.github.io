---
layout: manual
title: SQL Functions
headtitle: SQL Functions -
---

Conversion Functions
--------------------

nvl(anOption, aNonOption)

nvl takes a nullable (Option\[\]) as first parameter, and returns it
when not null, otherwise it returns the second (non nullable) argument.

Operators
---------

#### Boolean :

not, isNull, isNotNull, between, ===, \<, lt, \>, gt, \<=, lte, gte,
\<\>, exists, notExists, in, notIn

#### Math :

plus, +, minus, -, times, \*, div, /

#### String :

\|\| (concatenation), lower, upper, like, regex

Aggregate Functions
-------------------

max, min, sum, avg, sDevPopulation, sDevSample, varPopulation,
varSample, count(cols**), countDistinct(cols**)
