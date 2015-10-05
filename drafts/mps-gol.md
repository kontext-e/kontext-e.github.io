---
layout: post
title: Game of Life in Java on Steroids
description: "Game of Life in Java on Steroids"
tags: [Game of Life, MPS, Java, Language Workbench, Software Craftsmanship]
---


## Code Retreats and the Game of Life
[Code Retreats](http://coderetreat.org/) are a good way to improve our skills as
[Software Craftsmen](http://manifesto.softwarecraftsmanship.org/). Soon there is
the #gdcr15, reason enough to make the [Game of Life](https://en.wikipedia.org/wiki/Conway's_Game_of_Life)
a topic of a blog post.

Here is an example in Java:
![Game of Life in Java](/images/mps-gol.png)

## Java. Java? Java + extensions!
Hm, Java? That is not Java. Well, it looks like somewhat like Java, but that
tables, that colored constants, that initialization of the generation array,
that operations? It's not hard to notice: there are some extensions made to Java.
That takes for sure some years or at least months for a single person to create
such extensions! No, not really. Uh yeah, if you take the [OpenJDK](http://openjdk.java.net/)
and put your extensions there - but we don't. We take a [Language Workbench](http://www.martinfowler.com/articles/languageWorkbench.html).
To be more precise: we take the [MPS](https://www.jetbrains.com/mps/) Language Workbench.
Now it is easy to [modularize and put together](https://www.youtube.com/watch?v=lNMRMZk8KBE) programming languages.

In the following sections we will have a closer look on each of the new language concepts.
You can get the source [on my GitHub account](https://github.com/jensnerche/mps-gol).

## New Type: Coordinate

## Syntax sugar: alive and dead

## Decision Table

## Mapping Table

## Extending Operations: plus and minus

## Conclusions
