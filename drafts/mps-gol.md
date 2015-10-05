---
layout: post
title: bla
description: "bla"
tags: [en, jQAssistant, PlantUML, JaCoCo]
---


## Motivation
Let's assume you visited a [Code Retreat](http://coderetreat.org/), read [a book](http://www.amazon.com/Test-Driven-Development-Kent-Beck/dp/0321146530),
visited a workshop or a conference - or whatever. Now you are burning to introduce Unit Tests into your project - and hit the wall hard. Yes, you are
in a brownfield project. Who is not? Yes, there is no fairy offering you three wishes. Who saw one? So there is no easy way of transforming a 
we-are-not-writing-unit-tests-project into a TDD project. Where to begin? You could just starting with TDD from now on for every new line of code.
That is one way and not the worst one. (This would be to not doing Unit Tests.) But there is an alternative: find methods or classes in the project
which need Unit Tests most and start with them. It's not really hard to do that because [JaCoCo](http://eclemma.org/jacoco/) provides all you need:
the test coverage with covered and missed branches. There is an interesting correlation between the number of branches of a method and the test-me-begging.
Every branch adds a voice in the chorus singing test test me song. Now JaCoCo generates reports not only as HTML but also as CSV and XML. You could just
use a scripting language of your choice, invest some [Mana](https://en.wikipedia.org/wiki/Magic_%28gaming%29) and find what you are looking for.
Indeed I was quite successful with Groovy parsing the XML report, running in an ant build on the CI server as a watchdog for test coverage.

But technology gets better. Now there is no need for fiddling around with XML and scripts anymore - a simple database query does all the work.
And it offers also the great power of combining the test coverage with other metrics and static code analysis. In this article I'll explain the
technical side of getting started with Unit Tests in a brownfield project. The basis is once again [jQAssistant](http://jqassistant.org/). I assume
there is already a [maven](http://maven.apache.org) build in place, but not a single test yet. Then JaCoCo and the [Kontext E](http://www.kontext-e.de)
JaCoCo plug-in for jQAssistant were added to the project and some basic rules implemented.
