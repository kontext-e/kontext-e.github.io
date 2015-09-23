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
If there is no Unit Test yes, we need to set up the very basic configuration. In the pom, we need a dependency on JUnit:

            <dependency>
                    <groupId>junit</groupId>
                    <artifactId>junit</artifactId>
                    <version>4.12</version>
            </dependency>

and the surefire plugin:

                <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-surefire-plugin</artifactId>
                        <version>2.18</version>
                        <configuration>
                                <includes>
                                        <include>**/*Test.java</include>
                                </includes>
                        </configuration>
                        <dependencies>
                                <dependency>
                                        <groupId>org.apache.maven.surefire</groupId>
                                        <artifactId>surefire-junit47</artifactId>
                                        <version>2.18</version>
                                </dependency>
                        </dependencies>
                </plugin>

We create a new folder for our tests, configure the test source folder in the pom if it does not follow the convention
and create a dummy test which should fail:

    public class DummyTest {
        @Test
        public void thatShouldFail() throws Exception {
            Assert.assertTrue(1 > 2);
        }
    }

If we run 'mvn verify' we should see our test fail. If not, something is still wrong with our configuration which needs to be fixed.
Now we correct the test and celebrate our success. The first green bar!

Ok, now we are eager to write more tests for real production classes - but how to find the classes which deserve them most?
We add a test coverage checker.

# Add test coverage checker
The configuration of the JaCoCo test coverage checker is quite simple: we add the JaCoCo maven plugin:

                <plugin>
                        <groupId>org.jacoco</groupId>
                        <artifactId>jacoco-maven-plugin</artifactId>
                        <version>0.7.5.201505241946</version>
                        <executions>
                                <execution>
                                        <id>default-prepare-agent</id>
                                        <goals>
                                                <goal>prepare-agent</goal>
                                        </goals>
                                </execution>
                                <execution>
                                        <id>default-report</id>
                                        <phase>prepare-package</phase>
                                        <goals>
                                                <goal>report</goal>
                                        </goals>
                                </execution>
                                <execution>
                                        <id>default-check</id>
                                        <goals>
                                                <goal>check</goal>
                                        </goals>
                                        <configuration>
                                                <rules>
                                                </rules>
                                        </configuration>
                                </execution>
                        </executions>
                </plugin>

If we run now 'mvn verify', there should be a folder target/site/jacoco' created containing reports
as html, csv and xml. We can browse the html report and look for favourite units to test. Hm, what to look for?
One goal of Unit Tests is to prevent regression. Regression when we change something. Does regression occur on
one liners like getters and setters? Not so likely. Or is it raising it's ugly head in these monster methods
with lots and lots of branches? If we just add a little additional branch here and change that one there slightly?
Oh yes, most likely some corner case is broken now - or was always broken. So let's say for each method where the
number of branches exceeds a limit we need a certain test coverage. The first candidates are easy to spot and
we can [begin to write tests for legacy code](https://www.youtube.com/watch?t=883&v=_NnElPO5BU0).

For starting not bad, but in day to day work we do not want to check this manually. Or do we run our Continuous Integration
server for nothing? So let's automate this and introduce automatically checked rules.

# Add static analysis
Maybe you are already running jQAssistant for [keeping the architecture in sync with the documentation](http://techblog.kontext-e.de/keeping-architecture-and-doc-in-sync/)
or some other static analysis. If not you could at least risk a look on it - it's open and very flexible. That makes it very powerful.
We want to make use of this power.

Getting started is quite simple: follow the 'Maven' section of the [Get Started Guide](http://jqassistant.org/get-started/#).
Just like in the section for the first test, let the rule fail (e.g. by setting the WHERE clause to t.name =~ ".*TesTT") to see if all is working correctly.

Ok, but how do we bring the test coverage results and the jQA database together?

# Import test coverage into jQAssistant database using the Kontext E JaCoCo plug-in
This is also not hard. Just a little change here and there... Let's do this step by step.
First, we need to modify the pom. In the jqassistant-maven-plugin, we add a dependency for the 'jqassistant.plugin.jacoco':

                        <dependencies>
                                <dependency>
                                        <groupId>de.kontext-e.jqassistant.plugin</groupId>
                                        <artifactId>jqassistant.plugin.jacoco</artifactId>
                                        <version>1.0.0</version>
                                </dependency>
                        </dependencies>

and set a property with the name of the JaCoCo XML file:

                                                <scanProperties>
                                                        <jqassistant.plugin.jacoco.filename>jacoco.xml</jqassistant.plugin.jacoco.filename>
                                                </scanProperties>
                                                
and include target/site/jacoco into the set of scanned directories:

                                                <scanIncludes>
                                                        <scanInclude>
                                                                <path>target/site/jacoco</path>
                                                        </scanInclude>
                                                </scanIncludes>

so that the jqassistant-maven-plugin now looks like this:

                <plugin>
                        <groupId>com.buschmais.jqassistant.scm</groupId>
                        <artifactId>jqassistant-maven-plugin</artifactId>
                        <executions>
                                <execution>
                                        <goals>
                                                <goal>scan</goal>
                                                <goal>analyze</goal>
                                        </goals>
                                        <configuration>
                                                <failOnViolations>true</failOnViolations>
                                                <scanProperties>
                                                        <jqassistant.plugin.jacoco.filename>jacoco.xml</jqassistant.plugin.jacoco.filename>
                                                </scanProperties>
                                                <scanIncludes>
                                                        <scanInclude>
                                                                <path>target/site/jacoco</path>
                                                        </scanInclude>
                                                </scanIncludes>
                                        </configuration>
                                </execution>
                        </executions>
                        <dependencies>
                                <dependency>
                                        <groupId>de.kontext-e.jqassistant.plugin</groupId>
                                        <artifactId>jqassistant.plugin.jacoco</artifactId>
                                        <version>1.0.0</version>
                                </dependency>
                        </dependencies>
                </plugin>

In our rules file (my-rules.xml if you followed the Geting Started Guide) we have to add some constraints for the test coverage:

    <constraint id="test:TestCoverageForLowComplexity">
        <description>...</description>
        <cypher><![CDATA[
        match (cl:JacocoClass)--(m:JacocoMethod)--(c:JacocoCounter {type: 'COMPLEXITY'})
        where c.missed+c.covered >= 2 and c.missed+c.covered <= 3 and not(m.signature ='boolean equals(java.lang.Object)') and not(m.signature ='int hashCode()')
        with m as method, cl.fqn as fqn, m.signature as signature, c.missed+c.covered as complexity
        match (m)--(branches:JacocoCounter {type: 'BRANCH'})
        where m=method and branches.covered*100/(branches.covered+branches.missed) < 0
        return fqn, signature, complexity, branches.covered*100/(branches.covered+branches.missed) as coverage
        ]]></cypher>
    </constraint>

    <constraint id="test:TestCoverageForMediumComplexity">
        <description>...</description>
        <cypher><![CDATA[
        match (cl:JacocoClass)--(m:JacocoMethod)--(c:JacocoCounter {type: 'COMPLEXITY'})
        where c.missed+c.covered >= 4 and c.missed+c.covered <= 5 and not(m.signature ='boolean equals(java.lang.Object)') and not(m.signature ='int hashCode()')
        with m as method, cl.fqn as fqn, m.signature as signature, c.missed+c.covered as complexity
        match (m)--(branches:JacocoCounter {type: 'BRANCH'})
        where m=method and branches.covered*100/(branches.covered+branches.missed) < 80
        return fqn, signature, complexity, branches.covered*100/(branches.covered+branches.missed) as coverage
        ]]></cypher>
    </constraint>

    <constraint id="test:TestCoverageForHighComplexity">
        <description>...</description>
        <cypher><![CDATA[
        MATCH (cl:JacocoClass)--(m:JacocoMethod)--(c:JacocoCounter {type: 'COMPLEXITY'})
        WHERE c.missed+c.covered > 5 AND NOT(m.signature ='boolean equals(java.lang.Object)') AND NOT(m.signature ='int hashCode()')
        WITH m AS method, cl.fqn AS fqn, m.signature AS signature, c.missed+c.covered AS complexity
        MATCH (m)--(branches:JacocoCounter {type: 'BRANCH'})
        WHERE m=method AND branches.covered*100/(branches.covered+branches.missed) < 90
        RETURN fqn, signature, complexity, branches.covered*100/(branches.covered+branches.missed) AS coverage
        ]]></cypher>
    </constraint>

and add the constraints to the default group:

    <group id="default">
        <includeConstraint refId="my-rules:TestClassName" />
        <includeConstraint refId="test:TestCoverageForLowComplexity" />
        <includeConstraint refId="test:TestCoverageForMediumComplexity" />
        <includeConstraint refId="test:TestCoverageForHighComplexity" />
    </group>

That's no magic. It's [Cypher](http://neo4j.com/docs/stable/cypher-query-lang.html), the query language of the [Neo4j database](http://neo4j.com/)
which is used by jQAssistant. 

And now, for now you don't need to understand every detail of that. The important thing is: I suggest to group the
methods by 'complexity'. The more branches a method has, the more complex it is. Three complexity levels should be enough: low, medium, and high.
Methods with low complexity are so trivial (getters, setters) that tests would bring no gain. No coverage is needed. Then there
are the medium and high ones. What medium and high means in your project - its up to you to find adequate numbers. In this example let's start
with four and five branches for medium and higher than 5 for high complexity. 

Now we run 'mvn verify' again. And are overwhelmed. By the time that is consumed for the check. By the amount of missing tests. Well, not really
surprising for a fairly large codebase, written without any tests or other explicitly checked design constraints.

As a first aid to get back the control over the build time, let's deactivate the checks for low and medium complexity by commenting out the
'includeConstraint' tags in the group section. Methods with low complexity have no enforced test coverage anyway. And we change in the
rule for high complexity the

    WHERE c.missed+c.covered > 5
    
into

    WHERE c.missed+c.covered > 50

so that only the monsters show up. Let's run 'mvn verify'... better. Now you can go on and play with the numbers. You can also
add a 'SKIP x' to the rule and replace x by a certain number. That ignores the first x results so that the check gets passed. Why would we
do this? In that case we could also deactivate the rule completely, no? Not really. With this query we express more or less the following:
we have a test coverage as a target, but the current code base is as bad as it is and we accept it. But we do not want to make it even worse,
so every additional method with a too high complexity or too less test coverage gets reported. You can do a similar thing also with some

    AND NOT cl.fqn =~'com.example.module.*'

after the first WHERE and before the WITH to exclude packages or classes. By the way: the 'equals' and 'hashCode' methods are excluded
by default because in most cases they are complex but generated by the IDE. If you did a code review and come to the conclusion that
the reviewed methods is complex but very readable and not really error prone, you should exclude them in the same way. 


# Improve the coverage rules
* just declare the complexity ranges and their coverage thresholds

# Where to go
## Add other dependencies for tests
* hamcrest matcher
* mocking framework
* mockito
* jmockit if the code is very bad

## Iterate and improve
* for every iteration, play again with the numbers and/or excluded packages and classes to find a new set of testworthy methods

## Tackle the non-technical problems
* resistence by team members
* resistence by management

## Other jQA plugins
* using the git meta data
* employ FindBugs
* use also Checkstyle
* ask PMD for an opinion
* define some architectural documentation with PlantUML and check the consistency with the real architecture
