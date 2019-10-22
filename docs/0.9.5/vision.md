---
layout: manual
title: Design principles
headtitle: Vision - 
---

<h3 class='zquote' align='left'>

“It seems that perfection is reached not when there is nothing left to
add, but when there is nothing left to remove.”

</h3>
<p align='right'>

*Antoine de Saint-Exupéry*

</p>
<!-- <h3 align='center'>Minimalism in all its forms</h3> -->

### Minimalism in all its forms

-   What is left out of a framework or language is as important as what
    is included. It’s easy to pile on features, removing them is *very*
    hard, and any system will eventually evolve to a point of *feature
    saturation* i.e. the point where every new feature added has
    undesirable interference with existing ones, or the complexity is no
    longer justified by the functionality it provides. We could describe
    this state as *software senility*. Only by putting a lot of efforts
    and thoughts into how to add *as little as possible* into Squeryl
    will it have a healthy longevity.

<!-- -->

-   The functionality that is added should have the smallest conceptual
    *surface area* as possible. <br/>Lets define the ***feature
    density*** by this ratio : functionality / (conceptual *surface
    area*).  
    The *conceptual surface area* here means complexity, in terms of API
    size, number of concepts that need to be introduced. While
    complexity is certainly a subjective and imprecise concept, extreme
    *bloatware* is usually unanimously recognized as such, there is
    hardly any one for example that could claim that EJB2.0 has a high
    *feature density*.

<!-- than if it needs a new keyword, operator or API method. A good example of this is the composability of statements and type arithmetic : they cause *zero* increase of the API nor do they need to introduce special keywords, all the complexity is _hidden_ in the Squeryl's engine room. In contrast, the *forUpdate* method in Query[T] increases the _surface area_, but was deemed useful enough to deserve its presence. -->

-   Zero tolerance for redundancy : the worst form of redundancy is the
    one imposed by a framework or library on user code, followed by
    redundancy in the DSL.

<!-- the counterexample TIMTOWTDI(There is mode than one way to do it) -->

-   Keep in mid that a persistence layer addresses only one aspect among
    many in an application and should leave as much *conceptual room* as
    possible for the other aspects.

<!-- -->

-   Convention over Configuration.

<!-- Configurability adds flexibility, but it also adds ano -->
