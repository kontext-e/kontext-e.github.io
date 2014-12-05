---
layout: post
title: Plugins für jQAssistant
description: "Dieser Artikel stellt jQAssistant kurz vor und erläutert, welche Erweiterungen Kontext E bereitstellt."
tags: [jQAssistant]
---

Was ist jQAssistant?
--------------------

jQAssistant ist ein Werkzeug zur Sicherung der Codequalität. Es scannt die Artifakte des Projekts - Klassen, Property-Dateien, XML-Descriptoren usw. - und speichert die Daten in der Graphendatenbank Neo4J. Auf diesen Daten werden Analysen durchgeführt. Regeln können definiert und deren Einhaltung automatisch geprüft werden. Zum Beispiel können Namenskonventionen für Klassen, die Abhängigkeitsbeziehungen zwischen Paketen oder die Verwendung unerwünschter Klassen (z.B. java.util.Date) geprüft werden.  

Die Datenbank kann aber auch als Server gestartet werden. Mit einer Webschnittstelle kann man den Code erforschen oder neue Regeln ausprobieren.

Die Mächtigkeit dieses Ansatzes geht weit über bisherige Werkzeuge dieser Art hinaus. Eine Graphendatenbank ist hervorragend für diesen Zweck geeignet. Über die Neo4j-Abfragesprache "Cypher" ist es leicht möglich, Suchen zu definieren, die in einer IDE nicht möglich sind. Mittels Cypher werden auch die definierten Regeln geprüft. Alle Daten stehen dem Nutzer direkt zur Verfügung.

Entwickelt wird jQAssistant von der Firma Buschmais, ansässig in Dresden. Der initiale Commit auf GitHub erfolgte im März 2013. Auf Maven Central ist der Meilenstein 3 von Version 1.0 publiziert, auf GitHub kann man schon den Snapshot von Meilenstein 4 beziehen. Das Final Release wird nicht mehr lange auf sich warten lassen.

Aktuell gilt die Apache Licence 2.0. Es ist jedoch beabsichtigt, aus Lizenzkompatibilitätsgründen auf die GPL umzusteigen.

Weitere Metriken
----------------

Mittels der importierten Standarddaten kann man schon sehr gut strukturelle Prüfungen und Abfragen definieren. Im Buildprozess fallen aber noch eine Reihe weiterer Daten an. Es liegt nahe, auch diese in die jQAssistant-Datenbank zu importierten. Da jQAssistant explizit für Erweiterungen mittels Plugins entworfen wurde, ist das Importieren nicht schwer. Bei Kontext E haben wir deshalb jQAssistant-Plugins für die wichtigsten Testartefakte geschrieben.

Üblicherweise wird die Testabdeckung nach Zeilen und Zweigen für Unit Tests gemessen. Häufig werden damit auch Vorgaben verbunden, deren Verletzung den Buildprozess abbrechen. Wir verwenden jacoco, welches die Ergebnisse als XML exportieren kann. Diese XML-Datei lesen wir mit einem Scanner-Plugin ein. Einige Cypher-Statements stellen die Verbindungen zu den vorhandenen Klassen und Methoden her.

Sehr verbreitet ist die statische Codeanalyse mit FindBugs. Viele Fehlermuster werden damit gefunden. Auch FindBugs schreibt die Ergebnisse als XML, welches wir mit einem zweiten Scanner-Plugin einlesen. Die jQAssistant-Daten werden wieder mittels Cypher mit den FindBugs verbunden.

Als weiteres wertvolles Tool zur statischen Codeanalyse setzen wir Checkstyle ein. Es überrascht nicht, dass dieses eine XML-Datei schreibt, die wir einlesen und die Daten mit den vorhandenen verknüpfen.

Bisher haben wir die von den Werkzeugen ermittelten Metriken jeweils einzeln weiterverarbeitet. Mal mit XSLT, mal mit Groovy-Scripten. Damit wurden aus den Rohdaten Indikatoren gewonnen, die eine sinnvolle Codequalitätssicherung ermöglichen. Der CI-Server bricht den Build ab, wenn die Indikatoren auf eine zu schlechte Codequalität hindeuten.

Jetzt werden die Daten nicht mehr einzeln ausgewertet, sondern sind miteinander verbunden und an die Codestruktur gebunden. Das ermöglicht eine neue Dimension der Auswertung und die Verfeinerung der Indikatoren. Außerdem können neue Indikatoren eingeführt werden, die bisher mangels passender Datenlage nicht möglich waren.

Als Beispiel sei eine Codestelle genannt, die so geschrieben ist, wie eine vorgeschriebene Bibliothek es verlangt. Checkstyle und FindBugs warnen zu recht, aber die Codestelle ist sehr detailliert getestet. Damit wird sie akzeptabel.

Obwohl weder jQAssistant noch unsere Plugins final als 1.0-Version veröffentlich wurde, sind sie aber schon so stabil und funktionsreich, dass sie im täglichen produktiven Einsatz wertvolle Dienste leisten. Wenn der CI-Server sagt, dass an einer Stelle nachgebessert werden muss, dann ist das auch fast immer der Fall.

Stand der Implementierung
-------------------------

Die Plugins sind wie jQAssistant selbst in der Verwendung sehr stabil, aber die Programmierschnittstellen ändern sich noch. Das bezieht sich einerseits auf die Java-API, andererseits aber auch auf das Datenmodell in der Datenbank. Das bedeutet, dass nicht immer die letzten GitHub-Commits von jQAssistant und den Plugins miteinander kompatibel sind. Mit dem Erreichen der finalen Version 1.0 sind diese Probleme dann aber ausgeräumt und man kann beides als normale Dependency im Projekt eintragen.

Lizenz
------

Für die Kontext E Plugins für jQAssistant wurde noch keine Lizenz deklariert, d.h. vorerst unterliegt die Software dem gesetzlichen Urheberrechtsschutz. Vorgesehen ist aber, dass sie dieselbe Lizenz wie jQAssistant bekommen. Wie oben erwähnt ist die GPL zu erwarten, so dass auch die Plugins unter die GPL gestellt werden.

Beziehen und benutzen
---------------------

Die Kontext E jQAssistant Plugins werden auf GitHub gehostet. Bis eine offizielle Version über Maven Central beziehbar ist, muss man sich den Quellcode von [GitHub](https://github.com/kontext-e/jqassistant-plugins) besorgen und per gradlew selbst bauen. Voraussetzung ist, dass der aktuelle Snapshot von [jQAssistant] (http://jqassistant.org) im lokalen Maven Repository installiert ist.

Im Zielprojekt muss man dann jQAssistant und die Plugins als Dependencies eintragen. jQAssistant selbst kommt mit einer engen Maven-Integration. Kontext E hat einen Kommandozeilen-Runner gespendet. Somit kann man seine Projekte auch per IDE, mit einem Batch-Skript, per ant, gradle oder von anderen Tools und eigenen Projekten aus starten.
