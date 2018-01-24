---
layout: post
title: "Enrich scanned code with higher level concepts from Architecture Documentation"
description: "Enrich scanned code with higher level concepts from Architecture Documentation"
author: jens_nerche
tags: [en, jQAssistant, Neo4j, Asciidoc, arc42, Pattern]
---

There is the common saying "Only the code tells the truth."
Yes - but the code tells not the whole truth. It is not only
my personal opinion. I hear that often 
said by [Grady Booch](https://twitter.com/Grady_Booch) and
[Simon Brown](https://twitter.com/simonbrown). Therefore a little
bit of architecture documentation pays off sooner or later.

As my frequent blog
readers know, I like the toolchain of jQAssistant, Neo4j, Asciidoc, 
PlantUML, and arc42. So when I already scanned the project into
a Neo4j database and if there is more truth in the arc42 
architecture documentation, it can't be that hard to enrich the
graph with information from the documentation, no?

The idea of ["Building Higher-Level Abstractions of Source Code"](https://www.feststelltaste.de/building-higher-level-abstractions-of-source-code/)
or using [Java Packages as layers/modules/components for analyzing dependencies
between business subdomains](https://www.feststelltaste.de/analyze-dependencies-between-business-subdomains/)
is one key point of jQAssistant. My pain point with the "traditional"
approach is, that the technical thing of using Cypher to manipulate the graph
and the "business" thing (ok, here the business is architecture validation which
is also quite technical, hence the quotation marks) are mixed. Let me quote
[Markus Harrer](https://twitter.com/feststelltaste) here to illustrate that. In his 
awesome post about 
["Building Higher-Level Abstractions of Source Code"](https://www.feststelltaste.de/building-higher-level-abstractions-of-source-code/)
he adds the concept of "Subdomains" to the graph using this Cypher statement:

    UNWIND [
        { name: "Clinic" },
        { name: "Owner" },
        { name: "Person" }, 
        { name: "Pet" },
        { name: "Specialty" },
        { name: "Vet" }, 
        { name: "Visit" }
    ]
    AS properties
    CREATE (s:Subdomain) SET s = properties
    WITH s
        MATCH (t:Type)
            WHERE t.name CONTAINS s.name
        MERGE (t)-[:BELONGS_TO]->(s)
    RETURN s.name, t.name

Not very complicated and Cypher is nice to read, but I'm sure we can do better. 
To borrow some words from the Domain Specific Languages guys: 
We can stay in the language of describing architectures and make the language of 
Cypher and the Cypher query itself a reusable implementation detail.


## Example: Mark Used Design Patterns via Regular Expressions

I've chosen an example that is very similar to Markus', but a little bit more
fine grained: let's assume we are using a set of patterns in our design and
some classes that implement a certain pattern can be found via regular expressions.
In the simplest case a [Command pattern](https://en.wikipedia.org/wiki/Command_pattern)
has the suffix "Command", a [Factory pattern](https://en.wikipedia.org/wiki/Factory_(object-oriented_programming))
has the suffix "Factory" and so on.  
 
Let's also assume that the Pattern Language is defined and documented in Architecture Documentation
this way:

![table screenshot](/images/pattern_language/table_screenshot.png)

Now how can we set a Label of "Plugin" on all plugin classes, label command classes with "Command"
and so on? As you will see, it needs only three easy steps.

### First step: scan the documentation

As noted in the beginning, I like arc42 and Asciidoc, so the
[Plugin for reading Asciidoc documents](https://github.com/kontext-e/jqassistant-plugins/blob/master/asciidoc/src/main/asciidoc/asciidoc.adoc).
does the job here.
Note that this plugin uses the same version of AsciidoctorJ as jQAssistant does to
prevent runtime conflicts. I'm using here the current versions 1.3.0 of jQAssistant and
1.3.3 of the Asciidoc plugin. 

To use the Asciidoc plugin, add it as jQAssistant dependency:

*Maven*

```xml
    <dependency>
        <groupId>de.kontext-e.jqassistant.plugin</groupId>
        <artifactId>jqassistant.plugin.asciidoc</artifactId>
        <version>1.3.3</version>
    </dependency>
```

*Gradle*

```groovy
	jqaRt("de.kontext-e.jqassistant.plugin:jqassistant.plugin.asciidoc:1.3.3")
```

Once we brought the code and the documentation next to each other in the same database, we
are on a good way and ready for the next step:

### Second step: Finding the relevant information in the documentation

Since version 1.3.3 the Asciidoc plugin also imports attributes on tables, sections, and lists.
We can mark our table shown in the picture in the documentation with an attribute *label="Pattern"* 
Then the Asciidoc source looks like this:

    .Design Pattern of Classes Matched by Regular Expressions
    [options="header", label="Pattern"]
    |===
    | Regex                                 | Pattern
    | de.kontext_e.jqassistant.*Plugin      | Plugin
    | de.kontext_e.jqassistant.*Command     | Command
    | de.kontext_e.jqassistant.*Factory     | Factory
    |===

To make the live easier for later Cypher queries, let's add some labels to the table cells we
are interested in. The jQAssistant concept in Asciidoc form looks like this:

    
    [[structure:MarkAsciidocTypeRegex]]
    [source,cypher,role=concept]
    .Mark Asciidoc Table Cells that contain regular expressions for types that have to be marked with given labels.
    ----
        MATCH
            (a:Asciidoc:Table)-[:BODY]->(body),
            (a)-[:HAS_ATTRIBUTE]->(att:Asciidoc:Attribute)
        WHERE
            att.name='label' AND att.value='Pattern'
        WITH
            body
        MATCH
            (body)-[:CONTAINS_CELLS]->(regexCell:Asciidoc:Cell {colnumber: 0}),
            (body)-[:CONTAINS_CELLS]->(labelCell:Asciidoc:Cell {colnumber: 1})
        SET
            regexCell:RegularExpressionCell,
            labelCell:LabelCell
        CREATE UNIQUE
            (regexCell)-[:REGEX_FOR_LABEL]->(labelCell)
        RETURN
            regexCell, labelCell
    ----

Don't get confused here. There are two Asciidoc directories in this made up example project:

* one contains the arc42 architecture documentation of the project
* the other contains the jQAssistant concepts and constraints which could also be written in XML

So the table is located in arc42, the concept is located in jQAssistant document. You see that
now the technical thing and the "business" thing is not mixed anymore.

Ok, we are only one step away from our goal and need only to:

### Third step: Set the Pattern Names from Table as Labels

Well, there is a little obstacle
here: the names of the labels to be set are stored as properties in the nodes we labeled with "LabelCell"
in the query above. And we [can't set labels with property values](https://stackoverflow.com/questions/26536573/neo4j-how-to-set-label-with-property-value).
But a little known feature of jQAssistant comes to the rescue: jQAssistant allows the use of 
scripting languages in concepts. Wow! Did you know that? Awesome, the graph cannot only manipulated 
with Cypher, also JavaScript, Ruby, Groovy, whatever scripting language is available in the classpath
can be used. JavaScript works with every newer JDK out of the box, so let's use this and craft a 
jQAssistant concept:

    [[structure:LabelTypesMatchedByRegex]]
    [source,js,role=concept,requiresConcepts="structure:MarkAsciidocTypeRegex"]
    .Mark types with labels matched by regex.
    ----
        var graphDatabaseService = store.getGraphDatabaseService();
        // Define the columns returned by the constraint
        var columnNames = java.util.Arrays.asList("Type");
        // Define the list of rows returned by the constraint
        var rows = new java.util.ArrayList();
    
        var result = graphDatabaseService.execute("    MATCH\n" +
                                                           "        (type:Type),\n" +
                                                           "        (regexCell:RegularExpressionCell)-[:REGEX_FOR_LABEL]->(labelCell:LabelCell)\n" +
                                                           "    WHERE\n" +
                                                           "        type.fqn =~ regexCell.text\n" +
                                                           "    RETURN\n" +
                                                           "        type, labelCell.text as label\n");
    
        while(result.hasNext()) {
            var next = result.next();
            var node = next.get("type");
            var label = next.get("label");
            node.addLabel(org.neo4j.graphdb.DynamicLabel.label(label));
            var resultRow = new java.util.HashMap();
            resultRow.put("Class", node);
            rows.add(resultRow);
        }
    
        // Return the result
        var status = com.buschmais.jqassistant.core.analysis.api.Result.Status.SUCCESS;
        new com.buschmais.jqassistant.core.analysis.api.Result(rule, status, severity, columnNames, rows);
    ----

It boils down to two steps:

* find the Java Type nodes and Asciidoc table cells using a Cypher query
* iterate over the result and use node.addLabel to set the label at the Java Type nodes

Note that again the jQAssistant stuff does not contain any project specific information. As long
as some conventions are matched like marking the table with *label="Pattern"* the jQAssistant magic
can be reused in every project and all the relevant and project specific information is 
contained in the architecture documentation.
