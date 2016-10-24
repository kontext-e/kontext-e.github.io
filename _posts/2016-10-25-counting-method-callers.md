---
layout: post
title: Counting method callers
description: "How to use jQAssistant and Neo4j Cypher to count method callers."
author: jens_nerche
tags: [en, jQAssistant, Java, Software Architecture, Neo4j, Cypher]
---

In her exciting talk [Applying Java 8 Idioms to Existing Code](https://www.infoq.com/presentations/java8-lambda-streams), 
[Trisha Gee](https://trishagee.github.io/) states that demoing the refactoring to returning java.util.Optional
could be a mess because so much caller statements have to be changed. You may find yourself easily in a similar
situation every now and then if you have some spare time in the project and think by yourself: "Would'nt it be
nice to do a quick refactoring now?" If you decide to return an Optional and get 538 compiler errors because
of a wrong return type you can forget about 'quick'. So how do you find a nice spot fitting in your refactoring time box?

It is just one simple [Neo4j](https://neo4j.com/) [Cypher](http://neo4j.com/docs/developer-manual/current/) query 
if you are already using [jQAssistant](https://jqassistant.org/get-started/) in your project.
Let's suppose you want to know how many callers are of method located in the de.kontext_e.techblog.service package.
Here is the query:

```cypher
    MATCH 
        (caller:Method:Java)-[:INVOKES]->(callee:Method:Java)<-[:DECLARES]-(t:Type) 
    WHERE 
        t.fqn=~'de.kontext_e.techblog.service.*' 
    RETURN 
        t.fqn, callee.name, count(caller) AS callers
    ORDER BY 
        callers
```

That's it. Now you can assess the impact and choose wisely.
