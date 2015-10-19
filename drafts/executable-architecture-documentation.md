layout: post
title: Physical Units as extension of Java types
description: "Physical Units as extension of Java types"
author: jens_nerche
tags: [MPS, Java, Language Workbench, arc42, Asciidoc, PlantUML, Continuous Integration, jQAssistant, Architecture, Test]


## Current state
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
be written. There exist lots of markup languages which provide excellent rendering results, e.g. the most famous LaTeX.
But for my experiments I chose the new shooting star [Asciidoc](http://asciidoc.org/)

[MPS language for Asciidoc](https://github.com/kontext-e/mps-asciidoc)

### Join both
[MPS project for Executable Architecture Documentation](https://github.com/kontext-e/mps-ead)


### Use an accepted template for Architecture Documentation: arc42
There is a good template for this
known as [arc42](http://confluence.arc42.org/display/LANDINGZON/landing+zone). 

[Asciidoc version of arc42 template](https://github.com/arc42/arc42-template)


### Integrate into Gradle build
[Installing and Using the Asciidoctor Gradle Plugin](http://asciidoctor.org/docs/asciidoctor-gradle-plugin/)

[Asciidoctor Diagram](http://asciidoctor.org/docs/asciidoctor-diagram/)

[Gist with example of integrating Asciidoctor in Gradle](https://gist.github.com/aalmiray/7369b977a68baca32e13)
The tricky part: find versions that work together.

[Gradle JRuby plug-in](http://jruby-gradle.org)

Note if you are still forced to use JDK7:
In our beta testing of this feature, users on JDK7 may need to increase their available "PermGen" space for more complex projects via the gradle.properties setting of: org.gradle.jvmargs="-XX:MaxPermSize=512m"

### Execute architecture tests
[jQAssistant](http://jqassistant.org)

[Kontext E Plug-ins for jQAssistant](https://github.com/kontext-e/jqassistant-plugins)

[Neo4j](http://neo4j.com/)


## Vision
Give the software architects the most flexible and powerful tool for defining an architecture and crafting it into code.
The combination of MPS, jQAssistant, and Neo4j is the best match for this. 
