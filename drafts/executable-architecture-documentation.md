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
[MPS language for PlantUML](https://github.com/kontext-e/mps-plantuml)


### Bring the documentation to the same platform
[MPS language for Asciidoc](https://github.com/kontext-e/mps-asciidoc)

### Join both
[MPS project for Executable Architecture Documentation](https://github.com/kontext-e/mps-ead)

[asciidoc-plantuml](https://code.google.com/p/asciidoc-plantuml/)
AsciiDoc PlantUML filter plugin

### Use an accepted template for Architecture Documentation: arc42
[Asciidoc version of arc42 template](https://github.com/arc42/arc42-template)


### Execute
[jQAssistant](http://jqassistant.org)

[Kontext E Plug-ins for jQAssistant](https://github.com/kontext-e/jqassistant-plugins)

[Neo4j](http://neo4j.com/)


## Vision
Give the software architects the most flexible and powerful tool for defining an architecture and crafting it into code.
The combination of MPS, jQAssistant, and Neo4j is the best match for this. 
