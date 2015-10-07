---
layout: post
title: How to keep architecture and it's documentation in sync
description: "How to keep architecture and it's documentation in sync"
author: jens_nerche
tags: [en, jQAssistant, Neo4j, PlantUML, Asciidoc, XMI]
---

In software, there is always architecture. You can't have no architecture. 
But with it's documentation, there are two options: you can have one or not.
If you have one, there are two options: the documentation matches with the real architecture or not.
If your documentation matches with the real architecture, there are two options: it stays in sync or not.
If it should or has to stay in sync, there are two options: watch it manually or... automatically.
Automatically? Has Word(TM) a "Run all tests" button nowadays? No, it hasn't. And we don't use Word(TM).

But we use Jenkins as a Continuous Integration server. And we use [jQAssistant](http://jqassistant.org) with
some [additional plug-ins](http://techblog.kontext-e.de/jqassistant-plugins). With all of that in place only
a small additional effort is needed to check the consistency between architecture documentation and the coded one.

As friends of small but powerful tools we use [PlantUML](http://plantuml.com). There you can create class diagrams,
especially with packages. It's not needed to put classes into the packages, you can just use the packages to describe
the high level architecture. E.g. like this:

    @startuml
    
    skinparam packageStyle rect
    
    package de.kontext_e.jqassistant.plugin.plantuml.scanner {}
    package de.kontext_e.jqassistant.plugin.plantuml.store.descriptor {}
    
    de.kontext_e.jqassistant.plugin.plantuml.scanner --> de.kontext_e.jqassistant.plugin.plantuml.store.descriptor
    
    @enduml

With only a few lines of code, these diagram descriptions can be parsed in a jQAssistant plugin
[like this](https://github.com/kontext-e/jqassistant-plugins/tree/master/plantuml). It will be part of the next
release 1.1.0 of the Kontext E plugin suite for jQAssistant which will be compatible with jQAssistant 1.1.0 version.

Now we have the wanted... er documented and real architecture together in the same database. The last piece of the puzzle
is the jQAssistant rule which matches both of them:

    <constraint id="dependency:WrongDirection" severity="critical">
        <requiresConcept refId="dependency:Package"/>
        <description>Finds package dependencies which are in the wrong direction according to the documentation.</description>
        <cypher><![CDATA[
            MATCH
                (p1:PlantUmlPackage)-[:MAY_DEPEND_ON]->(p2:PlantUmlPackage),
                (p3:Package)-[:DEPENDS_ON]->(p4:Package)
            WHERE
                p1.fqn = p4.fqn
                AND p2.fqn = p3.fqn
            RETURN
                p3.fqn + "-->" + p4.fqn AS WrongDirection;
        ]]></cypher>
    </constraint>

Was not that hard, no? Ok, a few lines more and we can also scan [Asciidoc](http://asciidoctor.org) 
where PlantUML diagrams [are embedded](https://code.google.com/p/asciidoc-plantuml/wiki/Usage).
But that's... that is... that IS "Executable Architecture Documentation".

Not a friend of Ascii? So for jQA and Neo4j Cypher we can't help you, the rules are 
[Ascii art](http://jexp.de/blog/2014/06/geekout-software-analytics-with-graphs/).

But for the documentation part - well, of course you could use Word(TM) and a nice UML tool for the model pictures.
And then you export your pictures not only in png but also the models as [XMI](http://www.omg.org/spec/XMI/). With the
jQA standard XML plug-in and some Cypher magic or a dedicated XMI import plugin you also get the model into the database.

But these coarse grained building blocks are better managed by maven/gradle subprojects and/or splitted Spring contexts you ask?
Why this waste of energy, creativity, memory and computing power you ask? Yes of course you are right - for greenfield projects.
Consider a brownfield big ball of mud and you want to bring in structure and carve out modules. Then you are happy when you
can define rules and have such a powerful and flexible mechanism like jQA/Cypher queries to define exceptions from the rules
for the current state of code.

And these package dependencies may also be defined on a much more finer grained level for intra-module structure. You have maven
modules for your slices? And want to enforce the right access for in-slice layers? Or have additional rules for calls from a layer
of one slice to another layer of a different slice? (Plant-)UML and jQAssistant are your friends. 

Now you have two options: business as usual or make your lazy architecture documentation finally something useful for you.
