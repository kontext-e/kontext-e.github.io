layout: post
title: Physical Units as extension of Java types
description: "Physical Units as extension of Java types"
author: jens_nerche
tags: [MPS, Java, Language Workbench, arc42, Asciidoc, PlantUML, Continuous Integration, jQAssistant, Neo4j, Cypher, Architecture, Test]


## Previous state
In a former blog post
[How to keep architecture and it's documentation in sync](http://techblog.kontext-e.de/keeping-architecture-and-doc-in-sync/)
I described how you could 'execute' an architecture documentation as a test.

But the diagrams are just pictures (or text to be precise), not models. You cannot:
* navigate from one package in diagram A to the same package in diagram B just like in an IDE
* perform model validations
* define constraints
* execute refactorings (e.g. rename) like in an IDE, you have to do Find&Replace instead
* reference a class define in a class diagram in a sequence diagram
* reference artifacts of a diagram in the text as a real link between them
* find usages of diagram artifacts
* do lots of other interesting stuff

## Make it better

### Make a model of Package Diagrams
Not so surprising my first goal was to make a model out of the plain text. Being an MPS guy,
naturally my first choice was to create and use a Domain Specific Language and create textual models with it.
I re-used the [PlantUML](http://plantuml.com/) concrete syntax and published a first prototype of the
[MPS language for PlantUML](https://github.com/kontext-e/mps-plantuml) to GitHub.

But an architecture documentation does not only consist of diagrams. Equally important is a description
of the building blocks, discussions on design decisions and so on. 

### Bring the documentation to the same platform
The models and the documentation can only be combined if they are on the same platform. So I also had to bring
the documentation into the MPS PlantUML model. Therefore another DSL is needed in which the documentation can
be written. There exist lots of markup languages which provide excellent rendering results, e.g. the most famous (La)TeX.
But for my experiments I chose the new shooting star [Asciidoc](http://asciidoc.org/) for several reasons which 
become clear down below.

For this I created the [MPS language for Asciidoc](https://github.com/kontext-e/mps-asciidoc). Models of this
language were compiled to Asciidoc files which can be directly used by Asciidoc text processors like
[Asciidoctor](http://asciidoctor.org). A nice thing about Asciidoctor is that it can also handle embedded
diagrams using [Asciidoctor Diagram](http://asciidoctor.org/docs/asciidoctor-diagram/). Also PlantUML is supported.

### Template for Architecture Documentation: arc42
There is a good template for documenting a software architecture:
[arc42](http://confluence.arc42.org/display/LANDINGZON/landing+zone). 
In the download side there are several versions. But the most interesting resides on GitHub:
[Asciidoc version of arc42 template](https://github.com/arc42/arc42-template)
This one comes with a [Gradle](http://gradle.org) build configuration which makes other formats
like HTML, Markdown, and docx. Rumors say that also LaTeX may be supported in the future.

I imported the arc42 Asciidoc files into MPS in an Asciidoc model. Chapter 5 "Building Block View" is the one
where the package diagrams fit in. But there is also Chapter 9 "Design Decisions". Some Design Decisions
have consequences in the code that could be monitored easily. Let's take the example Logging Framework. After
discussing the alternatives and making a decision for e.g. log4j, a new design constraint has to be put in place
"do not use any class of java.util.logging".

### jQAssistant, Neo4j and Cypher
For this kind of rules was [jQAssistant](http://jqassistant.org) created. I wrote already 
[some posts](http://techblog.kontext-e.de/tags/#jQAssistant) about
it and also about the [Kontext E Plug-ins for jQAssistant](https://github.com/kontext-e/jqassistant-plugins).

jQAssistant is based on [Neo4j](http://neo4j.com/). The query language of Neo4j is [Cypher](http://neo4j.com/docs/stable/cypher-query-lang.html).
And - not hard to guess - I did the [MPS language for Cypher](https://github.com/jensnerche/mps-cypherquerylanguage).

### Join all
Until now there are a lot of more or less unrelated building bricks. To join them all, some mortar is needed. 
The mortar is the [MPS project for Executable Architecture Documentation](https://github.com/kontext-e/mps-ead).
It combines the MPS languages for Asciidoc, PlantUML, and Cypher. Now it is possible to integrate UML diagram models 
and Cypher query models in Asciidoc documentation models. To my knowledge, there is no other tool which combines
good text processing, UML modeling, a software structure database, and a database client. But that's exactly the
toolset needed by a software architect to define an architecture via models and description and check whether 
that architecture is reflected in the code. Leave one out and it doesn't really work. Models without describing 
text are just nice pictures. An architecture definition without automated checks on every build will sooner or later
diverge from the code.

Speaking about the automated checks: how did I achieve that?

### Integrate into Gradle build
As described above, the Asciidoc documents serve two purposes:

1. inform developers
2. contain rules for the code

So the integration into the [Gradle](http://gradle.org) build has also to do two things:

1. create some user friendly format like HTML or PDF
2. check the rules against the code

For the format conversion there is the
[Installing and Using the Asciidoctor Gradle Plugin](http://asciidoctor.org/docs/asciidoctor-gradle-plugin/)
guide and also a
[Gist with example of integrating Asciidoctor in Gradle](https://gist.github.com/aalmiray/7369b977a68baca32e13).
The tricky part is to find most versions that work together.

Note if you are still forced to use JDK7: you may need to increase their available "PermGen" space 
for more complex projects via the gradle.properties setting of: org.gradle.jvmargs="-XX:MaxPermSize=512m".

For the rule checking part there is jQAssistant with Kontext E plug-ins. jQAssistant can handle 
Cypher rules embedded in adoc files out of the box, so the design rules of chapter 9 were supported directly.
And for the chapter 5 part of the building blocks, I described in
[How to keep architecture and it's documentation in sync](http://techblog.kontext-e.de/keeping-architecture-and-doc-in-sync/)
how our PlantUML plug-in can be used.


### Current state
architecture documentation with tightly integrated UML models in the same documents
(did you ever try to modify a UML model embedded in an Office word processor?)
documentation is executable in the sense that (currently some important) rules can be checked in the build
process directly using the architecture documentation files

many building bricks
can be used in different configs, e.g. without MPS as text only 
all is open source or creative commons


## The Future
lots of work
make all the MPS languages better
gather hands-on experience in real life projects
find ways to check rules from other arc42 chapters
Execute architecture tests from within MPS just like unit tests were executed within an IDE - port that from the Executable Specification project

Give the software architects the most flexible and powerful tool for defining an architecture and crafting it into code.
The combination of MPS, jQAssistant, and Neo4j is the best match for this. 
Asciidoc, PlantUML, arc42

