---
layout: post
title: Unnecessary public visibility
description: "Finding methods which are public but are accessed only within the same package."
author: jens_nerche
tags: [en, jQAssistant, Java, Software Architecture, Neo4j, Cypher]
---

In her German book ["Langlebige Software-Architekturen"](http://www.llsa.de/) [Carola Lilienthal](https://www.wps.de/carola-lilienthal/)
tells the story of an architect who wants to know which public methods are currently _not_ called from outside the package (p. 117).

Sure, with a powerful tool like [Sotograph](https://www.hello2morrow.com/products/sotograph) she is using this is no problem - if you have
the money. But can you achieve this also with Open Source tools? Having read some of my previous posts (especially the ones about
[jQAssistant](http://techblog.kontext-e.de/tags/#jQAssistant)) you already know the answer: yes, of course! I'll show you how easy that is.

For this post, I created a little demo project with a Service and a Client calling this Service:

![Demo Project](/images/UnnecessaryPublicVisibilityProject.png)

The Service has two methods

```java
    public class Service {
    
        public void calledFromDifferentPackage(){
            onlyCalledInPackage();
        }
    
        public void onlyCalledInPackage(){}
    }
```

and the Client calls one of them from a different package

```java
    public class Client {
    
        public void call() {
            new Service().calledFromDifferentPackage();
        }
    }
```

I scanned the project [into a jQAssistant database and started the server for exploration](https://jqassistant.org/get-started/). 
Now I can query for the public methods not accessed from a different package in three easy steps.

First step: put a label 'Public' on the public methods

            MATCH
                (c:Type:Class)-[:DECLARES]->(m:Method)
            WHERE
                m.visibility='public'
            SET
                m:Public

Second step: put a label 'UsedFromDifferentPackage' on methods which are called from a different package

            MATCH
                (t1:Type)-[:DECLARES]->(m:Method),
                (t2:Type)-[:DECLARES]->(p:Method:Public),
                (package1:Package)-[:CONTAINS]->(t1),
                (package2:Package)-[:CONTAINS]->(t2),
                (m)-[:INVOKES]->(p)
            WHERE
                package1.fqn <> package2.fqn
            SET p:UsedFromDifferentPackage
    
Third step: query for the methods which have no label 'UsedFromDifferentPackage'

            MATCH
                (c:Type)-[:DECLARES]->(u:Method:Public)
            WHERE NOT
                u:UsedFromDifferentPackage
            RETURN
                c.fqn, u.name

Of course I could have done this in one more complex step. But I decided to separate the concerns in this way
because most likely I would add some WHERE clauses in the third step to exclude public APIs, unscanned entry points,
uninteresting packages, or examine only some submodules. As a nice side effect, the first two steps can be
easily transformed into jQAssistant concepts and the third one into a jQAssistant constraint.



I published the [source code on GitHub](https://github.com/kontext-e/UnnecessaryPublicVisibility).
