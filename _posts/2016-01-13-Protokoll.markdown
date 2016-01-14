---
layout: post
title:  "Protokoll vom 13.01.2016"
date:   2016-01-14 19:16:00 +0100
categories: allgemein protokolle
---
<b>Anwesende Personen:</b><br />
Natanael Arndt<br />
Robert Rößling<br />
Christian Frommert<br />
Elias Saalmann<br />
Arne Präger<br />
Maxi Bornmann<br />
Georg Hackel<br />
Simon Jakobi<br /><br />
Abwesend: Eric (entschuldigt)<br /><br />
Protokoll-Verantwortlicher: Christian Frommert<br />
<br />
<h2>Themen</h2>
* Recherchebericht<br />
* Genauere Funktionsübersicht<br />
* Vorprojekt<br />
* Qualitätssicherungskonzept<br />
* ToDo<br />
* Nächstes Treffen<br />
<br />
<h3>Recherchebericht</h3>
Gutes Feedback, kleinere Fehler im Dokument (Formatierung Tex), wird bei den nächsten Dokumenten nicht mehr vorkommen. 
Ansonsten Themen getroffen, knackig kurze Beschreibungen, super Sicht auf Konzepte, Aspekte vereint vorherige Punkte, 
schärft Projektziel und stellt Möglichkeiten dar. Team hat sich auf Nutzung von Jekyll (somit Ruby als Programmiersprache), 
Liquid als Templating-Sprache und RDF.rb als RDF-Library für Ruby geeinigt. RDF.rb kann mit SPARQL umgehen. Natanael hat nichts dagegen.
<br /><br />
<h3>Genauere Funktionsübersicht</h3>
Elias hat umfangreiches Fragepaket für Natanael zusammengestellt:<br />
<b>Für welche Ressourcen sollen HTML-Seiten gerendert werden?</b><br />
-> Alles was URI ist ist Ressource und soll gerendert werden (default)<br />
-> Durch Konfiguration soll man bestimmen können ob man innerhalb eines Namespaces bleibt oder kompletten Store durchforstet<br />
-> Beispiel: Select-Anweisung mit Parameter, zB s -> nur Subjekte, p -> nur Prädikate, o -> nur Objekte --> auch Konkatenationen sollen möglich sein, zB spo für alles<br />
-> Funktion für Suche nach / Select von URIs mit bestimmten Properties soll möglich sein<br /><br />
<b>Was passiert mit externen URIs?</b><br />
-> externe URIs sollen in Unterordnern abgelegt werden, zB http://www.example.org/Name soll auf Webserver unter <URL>/www/example/org/Name erreichbar sein<br />
-> <URL> soll dabei in Konfiguration hinterlegt werden (falls die Seite auf anderem Server deployed wird)<br />
-> Helper soll URIs umschreiben können<br />
<br /><br />
<b>Welche SPARQL-Anfragen sollenen unterstützt werden? (Select / Update / Construct / ..)</b><br />
-> MUSS: Select<br />
-> KANN: Construct (und weitere Verarbeitung durch Javascript)<br />
<br /><br />
<b>Wie sehen die Rückgabewerte einer SPARQL-Anfrage aus?</b><br />
-> Je nachdem wie die Anfrage war, kann 1 bis n Spalten haben (vgl. SQL)<br />
-> Somit verschiedene Tags nötigt, einmal für einzelne Rückgabe (eindeutiges Ergebnis) sowie mehrere -> Iteration (z.B. Listen)<br />
-> Rückgabetypen: URIs oder Literal/-e<br />
-> Wenn Literal zurückkommt soll man in der Abfrage spezifizieren welche Sprache ausgewählt werden soll (durch @en oder @de Tag gekennzeichnet)<br />
-> Weitere Datentypen möglich, zB Date (gekennzeichnet mit ^^xsd:date) oder xsd:double, müssen wir ebenso handeln können<br />
<br /><br />
Natanael stellte dann Beispiel vor:

{% highlight linenos %}
model: @prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

<http://example.org/Hans> <http://example.org/name> "Hans"@de, "Henry"@en, "Ans"@fr ;

                         <http://example.org/height> "1.0"^^xsd:double ;
                         <http://example.org/dateOfBirth> "2016-01-13"^^xsd:date .
<http://example.org/Anna> <http://example.org/name> "Anna"@de ;

                         <http://example.org/height> "1.0"^^xsd:double ;
                         <http://example.org/dateOfBirth> "2016-01-13"^^xsd:date ;
                         <http://example.org/kennt> <http://example.org/Hans> .
<http://example.org/Anna> a <http://example.org/Student> .

<http://example.org/Student> rdfs:subClassOf <http://example.org/Person> .
{% endhighlight %}
Bemerkung: Das 'a' ist äquivalent zu rdfs:subClassOf, somit ist Anna vom Typ Student und Student vom Typ Person, somit Anna vom Typ Person


{% highlight linenos %}
config: classes:

   "http://example.org/Person": "personTemplate"
   "http://xmlns.com/foaf/0.1/Person": "personTemplate"
instanceTemplates:

   "http://example.org/Hans": "professorTemplate"
defaultTemplate: "personTemplate"
{% endhighlight %}
Bemerkung: Hier sollen die zu verwendenden Templates aufgelistet werden. Evtl. in Config-Datei (.yml), inkl. Hierarchie. Dabei sollen die Super-Classes 
betrachtet werden (zB. Anna -> Student -> Student-Template? Wenn ja nutzen, wenn nein iterere -> subClassOf Person -> Person-Template? Wenn ja das nutzen, sonst weitersuchen


{% highlight linenos %}
template:

…

[SPARQL-Abfrage, zB ' select ?p ?o where { <resourceURI> ?p ?o } ' ]

…



Diese Seite gehört [[Template:property uri="http://example.org/name"]]!

er ist [[Template:property uri="http://example.org/height"]] Meter groß und wurde am {{ | property uri="http://example.org/dateOfBirth" }} geboren.

Er/Sie kennt [[Template:persionPartial]].

Er/Sie wird gekannt von [[Template:persionPartial]].
{% endhighlight %}
Bemerkung: Hier folgen dann die exakten Abfragen auf die SPARQL-Anfrage, die weiter oben spezifiziert wird. Wir arbeiten praktisch auf einer Abfrage und 
sollen dann direkt auf die Properties zugreifen können (-> Helpers)

<br /><br />
<b>Zu Helpers: Wie viele Kanten sollen diese bearbeiten können?</b><br />
-> Nur Kantenlänge 1, weiteres wenn benötigt / Zukunft / Kann-Ziel<br />
<br /><br />
<b>Welches Template soll benutzt werden wenn Instanz mehr als eine Klasse hat?</b><br />
-> Warnung ausgeben, erstes nutzen (welches in der Hierarchie höher steht)<br />
-> Es sollen auch Templates für einzelne Instanzen möglich sein, siehe oben Bsp: "http://example.org/Hans": "professorTemplate"<br />
-> Für Auswahl eines Templates sollten wir uns auch die <a href=https://github.com/AKSW/site.ontowiki/blob/2bf155662d0a709c004d4457fa5817be7ea5287d/helpers/Renderx.php target=_blank>Renderx.php</a> der aktuellen Lösung anschauen:  <br />
<br /><br />
<b>Welche Templates sollen mitgeliefert werden?</b><br />
-> Keine nötig, Nutzer muss selbst Templates anfertigen<br />
-> Für Demonstrationszwecke sollte jedoch Templates angefertigt werden, aber bitte nicht hart coden<br />
<br /><br />
<h3>Vorprojekt</h3>
Durch die sehr ausführliche Fragerunde konnte schon ein Vorprojekt abgestimmt werden: jekyll build soll auf ein Dataset ausgeführt werden und pro Subjekt 
jeweils eine HTML-Ressource generieren. Auf den HTML-Seiten soll nichts spektakuläres dargestellt werden (z.B. nur der Name der URI). Kann-Ziel des Vorprojekts: 
Es soll bereits eine Query ausführen können. Natanael war mit Vorprojekt-Beschreibung einverstanden.<br /><br />
<h3>Qualitätssicherungskonzept</h3>
Maxi hat bereits einige Punkte zusammengetragen. Diese wurden diskutiert, Maxi wird einige Änderungen umsetzen. Wir sollten das Thema Continuous Integration in das 
Konzept aufnehmen (ebenfalls Thema für Eric -> Testkonzept). Frage: Gibt es einen Jekyll Styleguide? (-> Todo Recherche)<br /><br />
<h3>ToDo</h3>
* Team: Bearbeitung der Themen bis Sonntag abschließen (Abgabetermin Montag)<br />
* Christian: Doodle-Umfrage für 1. Meilenstein<br />
* Christian: Styleguide für Jekyll?<br />
<br />
<h3>Nächstes Treffen</h3>
* Montag, 18.01.2016, 15.00 Uhr Uni-Campus (danach Computerpool), kleine Runde um Arbeitsplan und Qualitätssicherungskonzept zu finalisieren<br />
* Weekly Scrum am Mittwoch fällt aus, dafür evtl. Freitag ab 14.00 Uhr 1. Meilenstein -> Christian setzt Doodle-Umfrage auf<br />
<br /><br />
Wenn kein Termin Freitag nachmittag zustande kommt wird 1. Meilenstein beim nächsten Scrum präsentiert
