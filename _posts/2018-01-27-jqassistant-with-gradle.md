layout: post
title: Use jQAssistant with Gradle (and other build tools)
description: "Setup a Gradle project to use jQAssistant and additional plugins."
author: jens_nerche
tags: [en, Gradle, jQAssistant]

Ultrashort TL;DR:

Copy [jqa.gradle](https://raw.githubusercontent.com/kontext-e/jqa-gradle/master/jqa.gradle) 
into your project root and add this line into your build.gradle:

    apply from: 'jqa.gradle'

Now you can do 

    ./gradlew clean check jqa 
    
in your project.

The following longer version explains why and how it works.

jQAssistant comes with an excellent Maven support. But does it mean that you can't use
it with other build tools? Of course you can use it also with Gradle. Or Ant. Or whatever
you are using. In this post I'll show you how.

At the [Get Started](https://jqassistant.org/get-started/) page you find only the options
"Command Line" and "Maven". Well, yes, we could use the "Run Process" capability of the
build tool to start the jqassistant.cmd or jqassistant.sh. But there is also the artifact
[jqassistant-commandline available at Maven Central](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22jqassistant-commandline%22).
Using that one I'll show you the example of a Gradle integration.

Given you have already a build.gradle file, first you add some properties to make
version handling easier:

```groovy
project.ext["jqaversion"] = "1.3.0"
project.ext["jqapluginversion"] = "1.3"
project.ext["kejqapluginversion"] = "1.3.3"
```

The "jqaversion" is for the jQAssistant Core Framework. With a high probability you also want to
use some plugins, e.g. for scanning Java class files. For the plugins is the "jqapluginversion".
And in this example let's use also a plugin from an external contributor, so we need also the
"kejqapluginversion".

It is not really necessary, but for the reason of Separations of Concerns, I always like to
define a separate "configuration" for jQAssistant dependencies:

```groovy
configurations {
  jqa
}
```

Now let's use this configuration to declare the dependencies:

```groovy
dependencies {
  jqa("com.buschmais.jqassistant:jqassistant-commandline:${project.jqaversion}") {
    // because jQA 1.3 comes with Neo4j 2 and 3 support, there would be a classpath conflict
    exclude module: 'neo4j'
  }

  // list here all the plugins you want to use
  jqa("com.buschmais.jqassistant.plugin:java:${project.jqapluginversion}")
  jqa("com.buschmais.jqassistant.plugin:junit:${project.jqapluginversion}")

  // and plugins from other contributors
  jqa("de.kontext-e.jqassistant.plugin:jqassistant.plugin.git:${project.kejqapluginversion}")
}
```

And everything is set up to scan the project. To do so let's define a new task:

```groovy
task(jqascan, type: JavaExec) {
    main = 'com.buschmais.jqassistant.commandline.Main'
    classpath = configurations.jqa
    args 'scan'
    args '-f'

    args 'java:classpath::build/classes/main'
    args 'java:classpath::build/classes/test'
}
```

As you see in the task a Java program gets executed. We have to give the 
classpath because we defined a separate configuration, the class containing
the main method and the parameters as described in the 
[jQAssistant documentation](http://buschmais.github.io/jqassistant/doc/1.3.0/)

Finally it's time to scan your project:

    ./gradlew clean check jqascan

The same way works defining a task for applying concepts and checking constraints:

```groovy
task(jqaanalyze, type: JavaExec) {
  classpath = configurations.jqa
  main = 'com.buschmais.jqassistant.commandline.Main'
  args 'analyze'
}
```

Don't forget to put some rules in your projects jqassistant/rules folder as described
in the [Get Started](https://jqassistant.org/get-started/) and 
[jQAssistant documentation](http://buschmais.github.io/jqassistant/doc/1.3.0/)

and try 

    ./gradlew jqaanalyze

Maybe you also want to start the server to explore your application. Let's also define
a task for this:

```groovy
task(jqs, type: JavaExec) {
  classpath = configurations.jqa

  main = 'com.buschmais.jqassistant.commandline.Main'
  args 'server'

  standardInput = System.in
}
```

The jQA command line runs the server and prints "ServerTask - Press <Enter> to finish."
as the last line. To be able to Press <Enter> we need to declare the

    standardInput = System.in

as the last line in this task.

Basically that's it. You may find it useful to define more tasks for the other
command line commands like available-rules, available-rules, effective-rules and so on.
Especially cleanup is useful. Because resetting the store may take a while, I found it
useful to simply delete the store and report folders with this task:

```groovy
task jqaclean(type: Delete) {
    delete 'jqassistant/report'
    delete 'jqassistant/store'
}
```

and execute this task also in the standard clean task:

    clean.dependsOn jqaclean

But wait... what if I have a multi-module project? Do I have to list every of
my subprojects here? And didn't I also promise to use a third party plugin? 
At least the Git plugin is specified as dependency!

Ok, ok, here is a task definition for scan which is slightly longer:

```groovy
task(jqascan, dependsOn: ['jqaclean'], type: JavaExec) {
  classpath = configurations.jqa
  main = 'com.buschmais.jqassistant.commandline.Main'
  args 'scan'
  args '-f'

  rootProject.subprojects {
    args 'java:classpath::'+it.name+'/build/classes/main'
    args 'java:classpath::'+it.name+'/build/classes/test'
  }

  args '.git'
}
```

This scans all subprojects and also the Git repository. If you may ask yourself here
why you should scan the Git repo and what it has to do with a "Java Quality Assistant",
you find the answer in
[this excellent blog post](https://jqassistant.org/shadows-of-the-past-analysis-of-git-repositories/)


Here is the complete code with some additional convenciece tasks:

```groovy
project.ext["jqaversion"] = "1.3.0"
project.ext["jqapluginversion"] = "1.3"
project.ext["kejqapluginversion"] = "1.3.3"

configurations {
    jqa
}

dependencies {
    jqa("com.buschmais.jqassistant:jqassistant-commandline:${project.jqaversion}") {
        // because jQA 1.3 comes with Neo4j 2 and 3 support, there would be a classpath conflict
        exclude module: 'neo4j'
    }

    // list here all the plugins you want to use
    jqa("com.buschmais.jqassistant.plugin:java:${project.jqapluginversion}")
    jqa("com.buschmais.jqassistant.plugin:junit:${project.jqapluginversion}")

    // and plugins from other contributors
    jqa("de.kontext-e.jqassistant.plugin:jqassistant.plugin.git:${project.kejqapluginversion}")
}

task jqaclean(type: Delete) {
    delete 'jqassistant/report'
    delete 'jqassistant/store'
}

task(jqascan, dependsOn: 'jqaclean', type: JavaExec) {
    main = 'com.buschmais.jqassistant.commandline.Main'
    classpath = configurations.jqa

    main = 'com.buschmais.jqassistant.commandline.Main'
    args 'scan'
    args '-f'

    args 'java:classpath::build/classes/main'
    args 'java:classpath::build/classes/test'

    /* in a multi subprojects project it would be:
    rootProject.subprojects {
        args 'java:classpath::'+it.name+'/build/classes/main'
        args 'java:classpath::'+it.name+'/build/classes/test'
    }
    */

    args '.git'
}

task(jqaanalyze, type: JavaExec) {
    classpath = configurations.jqa

    main = 'com.buschmais.jqassistant.commandline.Main'
    args 'analyze'
}

task(jqaavailablescopes, type: JavaExec) {
    classpath = configurations.jqa

    main = 'com.buschmais.jqassistant.commandline.Main'
    args 'available-scopes'
}

task(jqareset, type: JavaExec) {
    classpath = configurations.jqa

    main = 'com.buschmais.jqassistant.commandline.Main'
    args 'reset'
}

task(jqaeffectiverules, type: JavaExec) {
    classpath = configurations.jqa

    main = 'com.buschmais.jqassistant.commandline.Main'
    args 'effective-rules'
}

task(jqaavailablerules, type: JavaExec) {
    classpath = configurations.jqa

    main = 'com.buschmais.jqassistant.commandline.Main'
    args 'available-rules'
}

task(jqareport, type: JavaExec) {
    classpath = configurations.jqa

    main = 'com.buschmais.jqassistant.commandline.Main'
    args 'report'
}

task(jqa, dependsOn: ['jqascan','jqaanalyze']) {
    jqaanalyze.mustRunAfter jqascan
}

task(jqs, type: JavaExec) {
    classpath = configurations.jqa

    main = 'com.buschmais.jqassistant.commandline.Main'
    args 'server'

    standardInput = System.in
}

clean.dependsOn jqaclean
```

If you put it into a (reusable) 
[jqa.gradle](https://raw.githubusercontent.com/kontext-e/jqa-gradle/master/jqa.gradle) 
file, you only need one additional line in your build.gradle:

    apply from: 'jqa.gradle'

You find a working [Gradle Blueprint on GitHub](https://github.com/kontext-e/jqa-gradle)
