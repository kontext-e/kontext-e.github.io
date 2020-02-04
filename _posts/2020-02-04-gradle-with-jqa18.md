---
layout: post
title: Using jQAssistant 1.8 with APOC plugin and Gradle
description: "Gradle dependency configuration when using jQAssistant 1.8 and the APOC plugin"
author: jens_nerche
tags: [en, jQAssistant, Gradle]
---

In case you are using
[jQAssistant with Gradle](http://techblog.kontext-e.de/jqassistant-with-gradle/)
I described a while ago, you may encounter an exception like this after upgrading to jQA 1.8 and using the APOC plugin:

```
java.lang.NoClassDefFoundError:
org/neo4j/driver/v1/Logging
        at java.base/java.lang.Class.getDeclaredMethods0(Native Method)
        at java.base/java.lang.Class.privateGetDeclaredMethods(Class.java:3167)
        at java.base/java.lang.Class.getDeclaredMethods(Class.java:2310)
        at org.neo4j.kernel.impl.proc.ReflectiveProcedureCompiler.compileProcedure(ReflectiveProcedureCompiler.java:227)
        at org.neo4j.kernel.impl.proc.Procedures.registerProcedure(Procedures.java:168)
        at org.neo4j.kernel.impl.proc.Procedures.registerProcedure(Procedures.java:157)
        at org.neo4j.kernel.impl.proc.Procedures.registerProcedure(Procedures.java:147)
        at com.buschmais.jqassistant.neo4j.backend.neo4jv3.Neo4jV3CommunityNeoServer.initialize(Neo4jV3CommunityNeoServer.java:77)
        at com.buschmais.jqassistant.neo4j.backend.bootstrap.AbstractEmbeddedNeo4jServer.initialize(AbstractEmbeddedNeo4jServer.java:21)
        at com.buschmais.jqassistant.core.store.impl.EmbeddedGraphStore.initialize(EmbeddedGraphStore.java:71)
        at com.buschmais.jqassistant.core.store.impl.AbstractGraphStore.start(AbstractGraphStore.java:50)
        at com.buschmais.jqassistant.commandline.task.AbstractStoreTask.run(AbstractStoreTask.java:46)
        at com.buschmais.jqassistant.commandline.Main.executeTask(Main.java:254)
        at com.buschmais.jqassistant.commandline.Main.executeTasks(Main.java:203)
        at com.buschmais.jqassistant.commandline.Main.interpretCommandLine(Main.java:195)
        at com.buschmais.jqassistant.commandline.Main.run(Main.java:78)
        at com.buschmais.jqassistant.commandline.Main.main(Main.java:49)
Caused by: java.lang.ClassNotFoundException: org.neo4j.driver.v1.Logging
        at java.base/jdk.internal.loader.BuiltinClassLoader.loadClass(BuiltinClassLoader.java:583)
        at java.base/jdk.internal.loader.ClassLoaders$AppClassLoader.loadClass(ClassLoaders.java:178)
        at java.base/java.lang.ClassLoader.loadClass(ClassLoader.java:521)
        ... 17 more
```

This is because:
 
* jQA "store" depends on "xo.neo4j.remote" which depends on "org.neo4j.driver:neo4j-java-driver:4.0.0"
* "jqassistant-apoc-plugin" depends on "org.neo4j.procedure:apoc" which depends on "org.neo4j.driver:neo4j-java-driver:1.7.3"

jQAssistant itself still runs on Neo4j 3.5, so downgrading the neo4j-java-driver seems to be an option. 
That can be done like this:

```
  jqa("com.buschmais.jqassistant.cli:jqassistant-commandline-neo4jv3:${project.jqaversion}") {
    exclude module: 'asm'
  }
  jqa("org.neo4j.driver:neo4j-java-driver:1.7.5") {
    // to resolve incompatible driver versions between CLI and APOC
    force = true
  }
  jqa("org.jqassistant.contrib.plugin:jqassistant-apoc-plugin:${project.jqaversion}")

  // all your other plugins go here
```

The [blueprint for a Gradle project using jQAssistant](https://github.com/kontext-e/jqa-gradle/) was updated,
so this from the previous blog post still works:

Add this line into your build.gradle:

    apply from: 'https://raw.githubusercontent.com/kontext-e/jqa-gradle/master/jqa.gradle'

Now you can do 

    ./gradlew clean check jqa 
    
in your project.


