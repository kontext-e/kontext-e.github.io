---
layout: post
title: Improved jQAssistant plugin for Gradle
description: "The new jQAssistant plugin for Gradle contains many improvements"
author: william_thimm
tags: [en, jQAssistant, gradle]
---

## Background

Jens once developed the de.kontext_e.jqassistant.gradle plugin that facilitates the use of jQAssistant through Gradle. However, this plugin began to show its age when newer Gradle versions no longer supported it, and the use of jQAssistant 2.1.0+ became impossible.

This was the catalyst for me to start updating and improving the plugin.

The old plugin worked by using Gradle's Request Handler. This worked fine; however nowadays, there are simpler approaches.
When I started working on the project I changed the structure of the plugin to use Gradle's JavaExec Tasks. This seemed like the perfect choice at the time as it provided everything I needed, and it required little configuration.
At its core version 2.X of the plugin declared the jQAssistant commandline tool as a dependency and executed the main class of jQAssistant with all the required and user specified arguments.
This change allowed me to significantly reduce the complexity of the Plugin by, for example, halving the number of classes.

However, as I hinted earlier, it was not quite that simple. Because of the tight integration with Gradle, Gradle overtook the resolution of all dependencies to build the classpath. I already had some trouble with the classpath and the logger of jQAssistant. For some reason I did not receive an output. After some tweaking of the classpath, I finally got it to work, thinking this would be the end of the classpath issues. This was until I started getting issues from developers using the plugin, regarding the classpath. Some reported troubles with missing classes, others pointed out that it was no longer possible to use plugins for neo4j itself, as it uses a different discovery mechanism to load plugins.

## Current State of the Plugin

After digging down into the causes of these problems and discovering that there were no quick fixes for them, I decided to rewrite much of the plugin. I had noticed over the course of the development of this plugin that it is best to keep dependency on Gradle to a minimum. The then obvious way was to launch jQAssistant using the jqassistant.cmd packaged with the commandline distribution. This way I only needed to declare Gradle Exec Tasks and pass along any commandline arguments that jQAssistant supports (the number of which has luckily been drastically reduced with the launch of jQAssistant 2.1)

This immediately fixed all the classpath issues, and was just because of that worth it. It also did not take too long, because by now, I already had some experience with Gradle. Additionally, I was able to further reduce the complexity of the plugin as now most of the configuration can only be done using the jqassistant config file approach. This led to a reduced need to expose configuration option and this removed the need for all the code configuring jQAssistant through Gradle. 

The entire plugin now basically only takes the configured options from the build.Gradle file and passes them onto the commandline distribution along the specified tasks. The only meaningful logic left in the plugin is the automatic adding of java source code to the scan task. This is a convenience feature that was in place from nearly the start, and I wanted to preserve it, as it can get otherwise pretty ugly to add all sources manually. (*)

Except that is not entirely accurate. There is one Task that I still have to address. As Gradle now no longer takes care of resolving the dependencies, the plugin needs to take care of that. This is the reason the installJQA task now exists. This task checks if there is already a jQAssistant installation present in the configured location, and if not, downloads the CLI and extracts it there. It is only necessary to run this task if there is no jQAssistant installation present, so in most cases, only once. The only time you would need to run this task again is when you want to change the installed version of jQAssistant. (in this case you need to delete the existing jQAssistant installation first. Just make sure you don't accidentally delete your rules and configuration as well)

I know this is technically a downgrade compared to the last version of this plugin, but I think it is a worthy trade-off for a clean and conflict-free classpath and full functionality of jQA. It even brings a small benefit: If you already have an installation fully configured and ready to go, you can just configure the location of it in the build.gradle and start using it :)

-------
(*) Unfortunately due to the nature of jQAssistant, this mechanism only works when there are no files and urls specified in the .jqassistant.yml, as otherwise, the commandline arguments get overwritten by them.

## How to use

Simply add the following plugin declaration to your build.gradle:

```groovy
plugins {
    id "de.kontext-e.jqassistant.gradle" version "3.0.0"
}
```

reload the Gradle project, and you should now have access to the jqassistant tasks. 

You can now configure jQAssistant using the jqassistant {} closure in your build.gradle.
The following options are listed below.

```groovy
jqassistant {
    // Allows for different jQAssistant version
    // only evaluated during installation Task
    toolVersion = 2.5.0

    // Allows for manual selection of neo4j version.
    // Available are only 4 or 5
    // Prioritises compatibility over user choice
    // If omitted, chosen default depends on current java version
    neo4jVersion 5

    // Allows for custom location of config file (default is shown below) (multiple possible)
    // absolute paths possible; relative paths are relative to jQAssistant install directory
    configFile ".jqassistant.yml"

    // Customize install location (default is shown below)
    // relative paths must be relative to the root directory of the project corresponding to this build.gradle
    // absolute paths are accepted as well
    installLocation './jqassistant'

    // Declare files/directories to be scanned, java sources are added automatically (multiple possible)
    // absolute paths possible; relative paths are relative to jQAssistant install directory
    // (will be ignored if scan directories are declared in config file;
    // automatic adding of java sources will be ignored as well)
    scanDir "directory/to/be/scanned"

    // Set additional args (multiple possible)
    option "-Djqassistant.scan.reset=true"
}
```

All of these options are optional and the defaults are shown above.

If the configured installLocation does not point to an already existing installation of jQAssistant, you now need to execute the installJQA task.

You now can use jQAssistant through Gradle :D

## More Information

For a more comprehensive guide on how to use the plugin have a look at the [repository of this plugin](https://github.com/kontext-e/jqassistant-gradle-plugin). It includes an example project where you can dig down into how the plugin works.

If you want to learn more about how to configure jQAssistant using the config files have a look at the [jQAssistant User Manual](https://jqassistant.github.io/jqassistant/current/#_yaml_files)

If you find any bugs or have feature requests, feel free to open an issue or a pull request.