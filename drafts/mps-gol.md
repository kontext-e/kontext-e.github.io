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
The Gol class starts like a quite normal Java class with a main method. But already the second method, run(), 
contains something special. As you see, Coordinate is like a build-in Java type. It could also be a class named
Coordinate, but there is a tiny difference: look at the initialization of the arraylist. These are no calls to
constructors, these are the notation (aka ["Concrete Syntax"](https://en.wikipedia.org/wiki/Parse_tree)) 
of a [Literal](https://en.wikipedia.org/wiki/Literal_%28computer_programming%29). Just like 5 for an integer or
"foo" for a string.

What do we need to create a new type? Surprisingly not that much. Of course the type itself:

![Coordinate Type](/images/gol/CoordinateType.png)

Note the little blue arrow. The CoordinateType extends something called "Type". This is the exact Type all 
other Java types like integer, long etc. extend too. Because of that our new CoordinateType fits nicely into
the existing Java type system.

Next there is the Literal:

![Coordinate Literal](/images/gol/CoordinateLiteral.png)

Note again the blue arrow. The CoordinateLiteral extends an Expression - yes, again the very same Expression
which is the base for all Java expressions.

Now we only have to tell the type system, that our CoordinateLiteral is of type CoordinateType:

![CoordinateLiteral is of type CoordinateType](/images/gol/typeof_CoordinateLiteral.png)

With some MPS typesystem syntax, this basically declares the the type as described above.

That's it. After five minutes or so we are able to use a new Java type. It does not yet do something useful,
therefore we have to declare the semantics via a generator. But that is the topic of a different section down below.

## Syntax sugar: alive and dead
In the next method, nextGeneration(...), a two colorful constants catch the eye. 'alive' and 'dead' are used like
constants or enums, but have a custom color. We could also assign other properties like underlined text, italic or bold font,
a different font size and so on. Can normal Java IDEs do that too? No - and perhaps that would be a nice issue to be filed
in Eclipse, NetBeans or Intellij IDEAs backlogs: assigning representation properties to constants and enums.

And it's really no big deal to get domain specific styled constants:

![AliveExpression](/images/gol/AliveExpression.png)

Again we extend an Expression. But that does not explain the different style of the text. This leads to a topic 
I did not mention so far: every new language concept needs or may declare how it is presented. Needs or may, eh?
Yes, if we inherit from a language concepts which provides a decent representation we can go with it. If we don't 
inherit or want to override the inherited style, we need to define a so called 'editor'. Let's have a look at the
editor for the alive expression:

![Alive Expression Editor](/images/gol/AliveExpression_Editor.png)

Hm, not that impressive. We see that it's the 'editor for concept AliveConcept'. And it's the default one. Yes,
we may declare more than one representation for a concept, e.g. for color-blind people. But let's focus for now on
only one editor. We also see that the #alias# should be shown. Alias? Yes, please go back to the Alive Expression picture.
There it is: 'alias: alive'. But how comes the color in? We can declare styles in a different tool window:

![Alive Expression Editor Style](AliveExpression_Editor_Inspector.png)

And the same goes for Dead Expression with the style 'text-foreground-color : red'.

## Decision Table

## Mapping Table

## Extending Operations: plus and minus

## Semantics: Generate Java code

## Conclusions
