---
layout: post
title: Kontext E jQAssistant Plugin Suite Relation Name Changes
description: "List of changes of relation names for our jQAssistant Plugin Suite"
author: jens_nerche
tags: [en, jQAssistant]
---

It was one small change from *1.3.x* to *1.4.0*, but unfortunately it may be 
a large change for you ()depending how many Cypher queries you already have):
to harmonize relation names with jQAssistant main distribution.

In jQAssistant main distribution all relation names are in singular. The Kontext E
plugins often used plural names. It felt natural for me because of the 1:N nature
of the relations, and I didn't notice until Dirk told me.

After some considerations I decided that a unique style of naming in the jQAssistant
world weights more than the inconvenience of changing existing queries. 
So I changed the plural names to singular as described in the following sections.


## :FindBugs:BugInstance
- CLASS -> HAS_CLASS
- METHODS -> HAS_METHOD
- FIELDS -> HAS_FIELD

## :FindBugs:SourceLineContainer
- SOURCELINE -> HAS_SOURCELINE
 
## :Asciidoc:List
- HAS_ITEMS -> HAS_ITEM

## :Asciidoc:Table
- COLUMNS -> HAS_COLUMN
- HEADER -> HAS_HEADER
- BODY -> HAS_ROW
- FOOTER -> HAS_FOOTER
 
## :Asciidoc:Row
- CONTAINS_CELLS -> CONTAINS_CELL

## :Checkstyle:File
- CHECKSTYLE_ERRORS -> CHECKSTYLE_ERROR

## :Checkstyle:Report
- CHECKSTYLE_FILES -> CHECKSTYLE_FILE

## :Jacoco:Class
- HAS_METHODS -> HAS_METHOD

## :Jacoco:Method
- HAS_COUNTERS -> HAS_COUNTER

## :Jacoco:Package
- HAS_CLASSES -> HAS_CLASS

## :Jacoco:Report
- HAS_PACKAGES -> HAS_PACKAGE

## :Plaintext:Directory
- HAS_FILES -> HAS_FILE
- HAS_DIRECTORIES -> HAS_DIRECTORY

## :Plaintext:File
- HAS_LINES -> HAS_LINE

## :PlantUml:Diagram
- CONTAINS_GROUPS -> CONTAINS_GROUP

## :PlantUml:File
- CONTAINS_DIAGRAMS -> CONTAINS_DIAGRAM

## :PlantUml:Group
- HAS_CHILD_GROUPS -> HAS_CHILD_GROUP
- HAS_LEAFS -> HAS_LEAF

## :Pmd:File
- HAS_VIOLATIONS -> HAS_VIOLATION

## :Pmd:Report
- HAS_FILES -> HAS_FILE
