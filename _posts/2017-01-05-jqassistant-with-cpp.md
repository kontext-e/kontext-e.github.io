---
layout: post
title: Combine jQAssistant with C++ (somehow)
description: "Quick and simple way to verify C++ module dependencies using jQAssistant"
author: jens_nerche
tags: [en, C++, jQAssistant]
---

In this post you can read that it may surprisingly quick and simple to make use of
jQAssistant for verifying module dependencies in C++.

Over the last year I was busy developing some C++ application. As usual it starts nice and clean,
gets into production - and evolves. Although I employed analyses tools like [Valgrind](http://valgrind.org)
and [CPPCheck](http://cppcheck.sourceforge.net/) I missed something like [jQAssistant](http://jqassistant.org)
to check architectural rules. As I updated the architecture documentation again I noticed that the thing
I really wanted was to check module dependencies. Sure, there are tools like [CppDepend](http://www.cppdepend.com/),
[Sonargraph](https://www.hello2morrow.com/products/sonargraph) etc., but I also wanted to take advantage
of the [Executable Architecture Documentation](http://techblog.kontext-e.de/keeping-architecture-and-doc-in-sync/)
described in a previous post.

My first attempt was to use the [Clang Tools](http://clang.llvm.org/docs/ClangTools.html) for reading the
source code and dump the [AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree) to a file. It turns out that
a C++ AST is considerably more complex than one may think in the first place. While I began to write a jQA plugin
for that, it may take a while - but I wanted something working _now_.

My second thought was to dump the [CLion](http://jetbrains.com/clion) PSI tree. But that would not work
for CI builds.

But finally I had a nice idea reading the [LLSA](http://www.llsa.de) book: Carola Lilienthal describes that
the Sotograph uses regular expressions to determine modules, patterns, layers etc. Because my project structure
is very simple - in the src folder are subfolders for each module, containing only files and no submodules - it
turns out that analyzing the #include declarations should be sufficient. As the Agile Principle says:
"the simplest thing that could possibly work". The effort for an experiment is very low, so I gave it a try
to find out if it could possibly work.

I created a "plaintext" plugin for jQAssistant which does exactly that: import the plain text line by line into
the jQA database. jQAssistant comes already with the notion of a "File" and a "Directory". So I can create
a relationship between two files which are connected by an #include with this Cypher statement:

```cypher
    MATCH
        (x:File:Plaintext), (d:Directory)-->(f:File:Plaintext)-->(l:Line:Plaintext)
    WHERE
        l.text=~'#include.*' and l.text=~'.*../.*' and d.fileName=~'/.*' and l.text=~('#include.*'+x.fileName+'.*')
    MERGE
        (f)-[:DEPENDS_ON]->(x)
    RETURN
        d.fileName, f.fileName, l.text, x.fileName
```

Next step is to connect the directories where the connected files are located:

```cypher
    MATCH
        (d1:Directory)-->(a), (d2:Directory)-->(b)
    WHERE
        (a)-[:DEPENDS_ON]->(b) and d1.fileName=~'/.*' and d2.fileName=~'/.*'
    MERGE
        (d1)-[:DEPENDS_ON]->(d2)
    RETURN
        a.fileName, b.fileName, d1.fileName, d2.fileName
```

But wait: I told you that I organized my source code so that one directory contains one module. So let's mark
the directories as modules:

```cypher
    MATCH
        (d:Directory)
    WHERE
        d.fileName=~'/.*'
    SET
        d:Module
    RETURN
        d.fileName
```

Now it is simple to find dependencies

```cypher
    MATCH
        (d1:Cpp:Module)-[:DEPENDS_ON]->(d2:Cpp:Module)
    RETURN
        d1.fileName, d2.fileName
    ORDER BY
        d1.fileName
```

or direct cycles

```cypher
    MATCH
        (d1:Cpp:Module)-[:DEPENDS_ON]->(d2:Cpp:Module)
    WHERE
        (d2)-[:DEPENDS_ON]->(d1)
    RETURN
        d1.fileName, d2.fileName
    ORDER BY
        d1.fileName
```

This works indeed astonishingly well for my purpose. The effort was really very low
because jQA brings a nice plugin concept and Neo4j Cypher supports the regular expressions.
I gained interesting insights into how the architecture developed and what unintended
dependencies I created while adding more features. Now it's time to pay back some
Technical Debt...
