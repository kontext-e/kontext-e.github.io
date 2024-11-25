---
layout: post
title: New jQAssistant C# Plugin
description: "Introduction to the new C# Plugin by Kontext-e"
author: william_thimm
tags: [en, jQAssistant, C#]
---

## About the Plugin

Given how successfully we use jqassistant here at kontext-e for our java projects, the need arose to extend jQAssistant.
I quickly discovered that there were already efforts made to analyze C# code using jQAssistant. The team [softviz-research](https://github.com/softvis-research) already started work on a [jqa-csharp-plugin](https://github.com/softvis-research/jqa-csharp-plugin). This was a great start as it already layed to foundation for how to analyze a language not natively supported by jqassistant and what tooling to use. The plugin uses a two-step approach:

**csharp-to-json-converter:** This tool is written in C# and uses the Microsoft C# Code Analysis Tool to analyze C# code and write the result into json files. This tool automatically downloaded and started by the jqa-csharp-plugin.

**jqa-csharp-plugin:** As the name suggests, this tool represents the actual jqassistant plugin. It detects if the scanned file or directory is a C# project, and if so, starts the csharp-to-json-converter. After the converter has finished, the plugin reads the json files, does a bit of post-processing and stores the datastructures into the neo4j database. 

However, the extent of the analyzed properties was not yet enough for us to properly analyze large projects and verify their architecture etc.
Given that the repository seemed to be abandoned (as there haven't been any updates to it in 4 years) I decided to fork it and start expanding the functionality.

118 Commits later this structure is still in use. The only thing, that fundamentally changed, is that the plugin can no longer scan C# files, but instead has to be given a *.sln(f)-file. This is because a proper .Net solution provides the C# Code Analysis Tool with significantly more information. 

Among the features added are:
* Support for Records and Structs
* Support for Properties and their accessors
* Support for Partial Classes and Methods
* Improved Analysis of invocations
* Upgraded to .NET 8
* Supports now JQAssistant Version 2.1.0

I have also significantly expanded the test suite by adding unit tests and expanding the integration tests. This plugin was also thoroughly tested manually on various internal projects.

There is also now a suite of concepts included with the plugin. Though certainly not enough to enforce architectures or to prevent code rot, they can still be used as a nice starting point or inspiration for a more project-specific ruleset.

Due to the complexity of C# there are likely still bugs in the project, so I would be very thankful if any bugs are reported. Just open an issue on the GitHub project.

## How to install

* Install Java 17 or higher from [Oracle](https://www.oracle.com/java/technologies/javase/jdk17-archive-downloads.html)
* Download the jQAssistant Commandline Interface Version 2.1.0 or higher from [Maven](https://repo1.maven.org/maven2/com/buschmais/jqassistant/cli/jqassistant-commandline-neo4jv5/2.5.0/jqassistant-commandline-neo4jv5-2.5.0-distribution.zip)
* Unpack the .zip-file to your desired installation directory
* Add the .jqassistant.yml-file according to the [jQAssistant User Manual](https://jqassistant.github.io/jqassistant/current/#_yaml_files)
* Add the following lines to the .jqassistant.yml

````yaml
jqassistant:
  plugins:
    - group-id: de.kontext-e.jqassistant.plugin
      artifact-id: jqassistant.plugin.csharp
      version: 0.3.2
````

## How to use
* add the Path to the solution file of your project to the 'included files'-section within the .jqassistant.yml
* navigate to the jqassistant.cmd file on your commandline and execute 

```sh
jqassistant.cmd scan analyze report
```

## Further Information
For more information see:
* [the jQAssistant User Manual](https://jqassistant.github.io/jqassistant/current)
* [the jqa-csharp-plugin repo by kontext-e](https://github.com/kontext-e/jqa-csharp-plugin)
* [the csharp-to-json-analyzer repo by kontext-e](https://github.com/kontext-e/csharp-to-json-converter)