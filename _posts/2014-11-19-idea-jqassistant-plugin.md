---
layout: post
title: An IntelliJ IDEA Plugin for jQAssistant
description: "Find classes using Neo4j Cypher queries against an existing jQAssistant database"
author: jens_nerche
tags: [en, jQAssistant, IDEA, Plugin]
---

Our German speaking readers already know from the last post that we contribute to the jQAssistant project.
We do so because we use [jQAssistant](http://jqassistant.org) in normal everyday work. There are a bunch of architecture and design rules
to keep our projects clean and in shape. 

From time to time there is the need to have a more powerful search mechanism than even the awesome ["Search Structurally"](https://www.jetbrains.com/idea/help/structural-search-and-replace-general-procedure.html)
feature of IntelliJ IDEA provides. Wouldn't it be nice to have the power of database queries to find classes? The good
news is: you have! Today we published our IntelliJ plugin to GitHub. You find it at [GitHub](https://github.com/kontext-e/idea-jqa-plugin)
It's licensed under the GPLv3. The bad news is: it's still an early version, so you don't find it in the JetBrains Plugin Repository.
You have to clone the project an build the plugin yourself. Please follow the instructions in the README file.

Feel free to file issues and send pull requests.
