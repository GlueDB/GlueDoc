# Schmierzettel

Auf diesem Schmierzettel notieren wir Informationen, die wir dann später ordentlich wegsortieren.

## Schnellstart

Damit du diese Anwendung weiterentwickeln kannst, brauchst du:

- alles, was im Dockerfile aufgeführt ist (noch aufschreiben)
- alles, was ist .gitlab-ci.yml aufgeführt ist (noch aufschreiben)

## Docker compose

- damit kann man mehrere Container starten und »orchestrieren«
	- Beschreibung in »docker-compose.yml«
	- Starten mit »sudo docker-compose up«
	- Bauen mit »sudo docker-compose build«

## Automatisierte Tests

### Unit-Tests

Die Unit-Tests liegen im Verzeichnis »t« und sind normale »Test::More«-basierte Testprogramme. Du kannst sie daher mit »prove« ausführen. Sie da sie die Module testen, die wir hier entwickeln, musst du die Testprogramme mit der Option »-l« aufrufen, damit das Verzeichnis »lib« automatisch in den Modulsuchpfad aufgenommen wird.

### Akzeptanztests

Die Akzeptanztests liegen im Verzeichnis »t/features«. Sie sind in der Sprache »Gherkin« geschrieben und mit dem Programm »pherkin«, das in der Distribution »Test::BDD::Cucumber« enthalten ist, ausgeführt.

Wie auch bei den Unit-Tests musst du die Option »-l« verwenden.

### Test der Distribution

Dist::Zilla testet die Distribution auf vielfältige Weise. Diese Tests rufst du mit »dzil test« auf.

### Aufruf aller Tests

Das Shell-Skript »run_tests.sh« ruft nacheinander folgende Tests auf:
- Unit-Tests
- Akzeptanztests
- Distributions-Tests

## Arbeiten mit Dist::Zilla

Technische Notiz: Die zitierte dist.ini am Anfang des Dokuments ist veraltet. Neu machen.

- "run_tests.sh" lässt er die Unit-Tests laufen, dann die Akzeptanztests, und dann alle von Dist::Zilla.
- die von Dist::Zilla kann man auch einzeln aufrufen: »dzil test --release« oder »dzil test --author«.
- Wenn die ohne Warnungen durchlaufen, können wir an ein Release denken. »dzil release« ...
  - bastelt ein tar.gz,
  - taggt den Branch mit der Version
  - und pusht den Kram sowohl zum origin (perlservices) als auch zu gitlab.

### Perlcritic und Dist::Zilla

#### TestingAndDebugging::RequireUseStrict

Diese Policy soll verhindern, dass kein "use strict" verwendet wird. Dass das sinnvoll ist, steht außer Frage. Es ist nun so, dass unser eigenes OTRStoolbox::Common die _strictures_ automatisch anschaltet, und Mojolicious ebenfalls. Die "use strict"-Direktive fehlt also in den Quellen, in denen diese Module eingebunden werden. Damit das Dist::Zilla-Plugin für Perlcritic schweigt, *muss* diese Richtlinie *direkt* nach der Package-Direktive ausgeschaltet werden - praktisch also in Zeile 2 der Quelle.

## Lauffähiges Docker-Image das Postmasterfilter-Debuggers bauen

Auf Gitlab gibt es das Projekt »pmfdb-image« (Postmasterfilterdebugger-Image). Das Projekt ist so eingerichtet, dass nach jedem Commit ein Docker-Image gebaut wird. Wenn aus diesem Image ein Container gestartet wird, ist der Postmasterfilterdebugger gestartet und kann auf Port 8080 angesprochen werden.

### Wie wird das Image gebaut?

Das Image basiert auf dem zur Build-Zeit aktuellen Docker-Image, das derzeit (Stand: Januar 2018) auf Alpine Linux basiert. In dieses System werden unsere Software-Pakete installiert, die beim Build aus den jeweiligen Gitlab-Projekten geholt werden. Das System wird so konfiguriert, dass beim Start sowohl der OTRS-Toolbox-REST-Server als auch die eigentliche Anwendung des Postmasterfilterdebuggers mit hypnotoad gestartet werden. Das fertige Image wird dann in die Gitlab-Container-Registry geschoben.

Dieses Projekt funktioniert genauso wie alle anderen Gitlab-Projekte. Wenn du also Genaueres zum Erstellen des Images wissen möchtest, kannst du es einfach in der Datei ».gitlab-ci.yml« nachschlagen.

### Kann ich das auch lokal bauen?

Ja, kannst du :-)

Der Build in Gitlab ruft nur Teile eines Makefiles auf, dass du auch lokal verwenden kannst. Dafür musst du folgende Umgebungsvariablen setzen:

- ENV am besten auf »dev«. Hiermit legst du fest, welches Skript für Umgebungsvariablen ausgeführt wird. Kann prinzipiell beliebige Werte annehmen, es muss dann nur eine entsprechende Datei im Verzeichnis vorhanden sein. Diese wird dann als »env.sh« in das Dateisystem eingeblendet.
- IMAGE_TAG auf das Tag, den das Image bekommen soll. Such dir etwas hübsches für den lokales System aus. In der Gitlab-CI-Beschreibung wird diese Variable auf einen Wert gesetzt, der dafür sorgt, dass das Image in die Gitlab-Container Registry geschoben werden kann.
- ARTIFACT_TOKEN auf den Wert, der in unseren Gitlab-Projekten unter diesem Namen als geheime Variable in den CI-Settings eingestellt ist. Bitte nirgendwo skripten oder sonstwo aufschreiben, da jeder, der dieses Token hat, sich als Gregor gegenüber Gitlab authentifizieren und das API benutzen kann.

Dann kannst du folgende Befehle sinnvoll verwenden:
- »make prepare« holt die benötigten Artefakte über die Gitlab-API und legt sie im lokalen Dateiverzeichnis ab.
- »make build« baut dann das Image. Es verlässt sich darauf, dass vorher »make prepare« ausgeführt wurde.
- »make shell« fährt den Container hoch und startet anstelle der Anwendung eine Bash.
- »make run« fährt einen Container hoch und startet dabei die Anwendung.

Hinweis: Ich habe »make prepare« und »make build« in zwei Schritte aufgeteilt, damit du die Möglichkeit hast, an den Quellen etwas zu ändern, bevor das Image gebaut wird.

