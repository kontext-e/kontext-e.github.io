* maven
* jQAssistant
* JaCoCo
* Kontext E JaCoCo Plugin for jQAssistant

# Motivation
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

# Find an example project
First let's find some example project for this exercise. You think there is no such project with a considerable size anymore? Not true mate, 
unfortunately there are enough of them. Otherwise this article would make no sense, no? So let's choose [PlantUML](http://sourceforge.net/projects/plantuml/).
It comes with some 2500 classes and not a single Unit Test. For playing around with it I checked out the source and put it into a 
[GitHub repo](https://github.com/jensnerche/plantuml).

Now we are ready for the real hands-on experience. We will add a rudimentary test infrastructure so that there is at least one Unit Test to go on with.
Than we add the static analysis and the test coverage checker. When both is in place, we bring it together in the same database and create coverage rules
using database queries. Last but not least we will have a look on various ways for further improvements.

Ok, let's begin!

# Add rudimentary test infrastructure
* pom: add a dependency for junit, plugins surefire and failsafe plugins
* pom: maybe configure test source folder if not in convention
* create a folder named 'test'
* put a dummy test in it (which should fail)
* run mvn verify and watch the test fail
* make the test pass (e.g. assertTrue(1 < 2)

# Add static analysis
* pom add jqassistant as plugin
* add jqassistant folder with a rule
* make the rule fail, e.g. TesTT
* make the rule pass again

# Add test coverage checker
* jacoco
* pom: add jacoco maven plugin
* run mvn verify
* target/site/jacoco should be there, including a jacoco.xml

# Import test coverage into jQAssistant database using the Kontext E JaCoCo plugin
* pom: modify the jqassistant plugin configuration
* pom: set property jqassistant.plugin.jacoco.filename=jacoco.xml
* pom: add target/site/jacoco as scanned directory
* rules.xml: add constraints for test coverage
* run mvn verify
* be overwhelmed by the flood of missing tests
* add a SKIP x to each rule
* maybe add an AND NOT cl.fqn =~'com.example.module.*' to exclude packages or classes
* play with the numbers to find some 10-20 classes where you could start with writing tests

# Add other dependencies for tests
* hamcrest matcher
* mocking framework
* mockito
* jmockit if the code is very bad

# Iterate and improve
* for every iteration, play again with the numbers and/or excluded packages and classes to find a new set of testworthy methods

# Tackle the non-technical problems
* resistence by team members
* resistence by management

# Where to go
* using the git meta data
* employ FindBugs
* use also Checkstyle
* ask PMD for an opinion
* define some architectural documentation with PlantUML and check the consistency with the real architecture
