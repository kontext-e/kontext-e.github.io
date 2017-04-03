---
layout: post
title: "Managing Technical Debt with arc42 and jQAssistant: Building Block Dependencies"
description: "Making the build green again although there are dependencies in the wrong direction - and document them in the architecture documentation"
author: jens_nerche
tags: [en, jQAssistant, Neo4j, PlantUML, Asciidoc, arc42, Maven, Gradle, Tutorial]
---

In the [last post](http://techblog.kontext-e.de/executable-architecture-revisited) I described in detail how to use
the same document to check the defined against the actual architecture. If you tried 
it out with the example project, you found some dependencies in the wrong direction.
And perhaps also in your own projects. This time I'll show you how to handle this
Technical Debt in a way that both the CI build gets green again and it is also documented
in the arc42 architecture documentation.

## Documenting the unwanted dependencies

To make it visible and clear that there is Technical Debt regarding building block
dependencies, it is a good idea to document the unwanted dependencies just below
the description of the building blocks and the desired dependencies. I used for
that an Asciidoc table like this:

```asciidoc
.Unwantend Module Dependencies
[options="header"]
|===
| From                          | To                          | What should be done
| de.kontext_e.demo.business    | de.kontext_e.demo.core      | Because ...; Todo: ...
| de.kontext_e.demo.exporter    | de.kontext_e.demo.facade    | Because ...; Todo: ...
| de.kontext_e.demo.exporter    | de.kontext_e.demo.processor | Because ...; Todo: ...
| de.kontext_e.demo.importer    | de.kontext_e.demo.parser    | Because ...; Todo: ...
|===
```

and it gets rendered this way:

![rendered Technical Debt table](/images/techdebtdep/rendered-technical-debt-table.png)

In a real project I found it useful the have a third column to describe why the wrong
dependency is currently there and to propose refactorings.

## Add Asciidoc documents to the jQAssistant database

The freshly released version 1.2.0 of the Kontext E jQAssistant plugin suite 
contains also a [Plugin for reading Asciidoc documents](https://github.com/kontext-e/jqassistant-plugins/blob/master/asciidoc/src/main/asciidoc/asciidoc.adoc).
Note that this plugin uses the same version of AsciidoctorJ as jQAssistant does to
prevent runtime conflicts. So this Plugin works only with 1.2+ versions of jQAssistant
because former versions use a too old AsciidoctorJ version.

To use the Asciidoc plugin, add it as jQAssistant dependency:

*Maven*

```xml
    <dependency>
        <groupId>de.kontext-e.jqassistant.plugin</groupId>
        <artifactId>jqassistant.plugin.asciidoc</artifactId>
        <version>${jqassistant.kePlugins.version}</version>
    </dependency>
```

*Gradle*

```groovy
	jqaRt("de.kontext-e.jqassistant.plugin:jqassistant.plugin.asciidoc:1.2.0")
```

The documentation folder is already part of the pathes to be scanned. If it works,
you should now have lots of :Asciidoc labeled nodes in your database.

## Add a jQAssistant Concept for the Technical Debt

Next thing is to mark dependencies between Java packages as Technical Debt.
So I first find our Asciidoc table which I titled "Unwantend Module Dependencies"
and use the body:

```cypher
MATCH 
    (a:Asciidoc:Table)-[:BODY]->(body)
WHERE 
    a.title='Unwantend Module Dependencies'
WITH 
    body
```
In the first two columns I put the source and target package names, so select them:

```cypher
MATCH 
    (c1:Asciidoc:Cell {colnumber: 0})<-[:CONTAINS_CELLS]-(body)-[:CONTAINS_CELLS]->(c2:Asciidoc:Cell {colnumber: 1})
WITH 
    c1, c2
```
Now I find the matching Java packages:

```cypher
MATCH
    (m1:Java:Package), (m2:Java:Package)
WHERE
    m1.fqn = c1.text
AND
    m2.fqn = c2.text
```

and create a relationship between them:

```cypher
MERGE
    (m1)-[:TECHNICAL_DEBT]->(m2)
RETURN
    m1, m2;
```

The resulting Cypher query is this:

```cypher
MATCH 
    (a:Asciidoc:Table)-[:BODY]->(body)
WHERE 
    a.title='Unwantend Module Dependencies'
WITH 
    body
MATCH 
    (c1:Asciidoc:Cell {colnumber: 0})<-[:CONTAINS_CELLS]-(body)-[:CONTAINS_CELLS]->(c2:Asciidoc:Cell {colnumber: 1})
WITH 
    c1, c2
MATCH
    (m1:Java:Package), (m2:Java:Package)
WHERE
    m1.fqn = c1.text
AND
    m2.fqn = c2.text
MERGE
    (m1)-[:TECHNICAL_DEBT]->(m2)
RETURN
    m1, m2;
```

Now I can put the query into the jQAssistant rules file. As usual I show examples for
Asciidoc as well as for XML.


*Asciidoc*

```asciidoc
[[documented:TechnicalDebt]]
.Creates a relationship between two Packages for Technical Debt.
[source,cypher,role=concept]
----
MATCH 
    (a:Asciidoc:Table)-[:BODY]->(body)
WHERE 
    a.title='Unwantend Module Dependencies'
WITH 
    body
MATCH 
    (c1:Asciidoc:Cell {colnumber: 0})<-[:CONTAINS_CELLS]-(body)-[:CONTAINS_CELLS]->(c2:Asciidoc:Cell {colnumber: 1})
WITH 
    c1, c2
MATCH
    (m1:Java:Package), (m2:Java:Package)
WHERE
    m1.fqn = c1.text
AND
    m2.fqn = c2.text
MERGE
    (m1)-[:TECHNICAL_DEBT]->(m2)
RETURN
    m1, m2;
----
```

*XML*

```xml
    <concept id="documented:TechnicalDebt">
        <description>
            Creates a relationship between two Packages for Technical Debt.
        </description>
        <cypher><![CDATA[
            MATCH 
                (a:Asciidoc:Table)-[:BODY]->(body)
            WHERE 
                a.title='Unwantend Module Dependencies'
            WITH 
                body
            MATCH 
                (c1:Asciidoc:Cell {colnumber: 0})
                <-[:CONTAINS_CELLS]-(body)-[:CONTAINS_CELLS]->
                (c2:Asciidoc:Cell {colnumber: 1})
            WITH 
                c1, c2
            MATCH
                (m1:Java:Package), (m2:Java:Package)
            WHERE
                m1.fqn = c1.text
            AND
                m2.fqn = c2.text
            MERGE
                (m1)-[:TECHNICAL_DEBT]->(m2)
            RETURN
                m1, m2;
        ]]></cypher>
    </concept>
```

## Modify the jQAssistant Constraint WrongDirection

I modified the Constraint WrongDirection in a way that it also requires "documented:TechnicalDebt"
and that the Cypher query excludes documented Technical Debt dependencies:

*Asciidoc*

```asciidoc
[[dependency:WrongDirection]]
.Finds package dependencies which are in the wrong direction according to the documentation.
[source,cypher,role=constraint,requiresConcepts="dependency:TransitivePackageDependencies, documented:TechnicalDebt",severity=critical]
----
MATCH
    (p1:PlantUml:Package)-[:MAY_DEPEND_ON]->(p2:PlantUml:Package),
    (p3:Java:Package)-[:DEPENDS_ON]->(p4:Java:Package)
WHERE
    p1.fqn = p4.fqn
    AND p2.fqn = p3.fqn
    AND NOT (p3)-[:TECHNICAL_DEBT]->(p4)
RETURN
    p3.fqn + "-->" + p4.fqn AS WrongDirection;
----
```

*XML*

```xml
    <constraint id="dependency:WrongDirection" severity="critical">
        <requiresConcept refId="dependency:Package"/>
        <requiresConcept refId="dependency:TransitivePackageDependencies"/>
        <requiresConcept refId="dependency:WrongDirection"/>
        <description>
            Finds package dependencies which are in the wrong direction 
            according to the documentation.
        </description>
        <cypher><![CDATA[
            MATCH
                (p1:PlantUml:Package)-[:MAY_DEPEND_ON]->(p2:PlantUml:Package),
                (p3:Java:Package)-[:DEPENDS_ON]->(p4:Java:Package)
            WHERE
                p1.fqn = p4.fqn
                AND p2.fqn = p3.fqn
                AND NOT (p3)-[:TECHNICAL_DEBT]->(p4)
            RETURN
                p3.fqn + "-->" + p4.fqn AS WrongDirection;
        ]]></cypher>
    </constraint>
```

## Running the checks again

If you run the checks in the example project again, the four constraint violations
should be gone.

## Some closing words

If dependencies in a project got never checked, there are most like unwanted dependencies.
Not surprising and it also hit me with the C++ project I [wrote about](http://techblog.kontext-e.de/jqassistant-with-cpp)
Breaking the build with any wrong dependency makes the build red for a long time.
Resolving those dependencies is not a five minute task. But documenting them and 
defining exceptions from the dependency architecture rules with exactly _the same table_
gives us a good chance to improve the architecture step by step over several iterations.


As you may have guessed from the title, more posts about handling Technical Debt
are in the pipeline. But before that, I have to introduce you to some other things first. 
