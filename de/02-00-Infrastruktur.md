# Infrastruktur

In diesem Kapitel erfährst du, aus welchen Bausteinen unsere Infrastruktur besteht.

## Sicht aus 10 km Höhe

Die Anwendung ist in Perl geschrieben und verwendet das Framework Mojolicious.

Der Aufbau der Quellen entspricht dem Perl-Standard. Ergänzend setzen wir Dist::Zilla ein, um die Anwendung möglichst komfortabel entwickeln zu können und sie prinzipiell wie eine Perl-Distribution auf CPAN veröffentlichen zu können.

Zur Unterstützung setzen wir Gitlab ein. Auf dieser Plattform werden nach jeder Code-Änderung die automatisierten Test ausgeführt. Außerdem werden automatisiert auszuliefernde Artefakte erstellt.

Grundlage der automatisierten Tests ist ein Ubuntu-basiertes System, das Gitlab für jeden Testlauf frisch installiert. Hierfür nutzt Gitlab ein Docker-Image, das Renée erstellt hat und alle von uns verwendeten Pakete enthält.

Die Dokumentation schreiben wir im Pandoc-Dialekt von Markdown. Mit »pandoc« generieren wir dieses Dokument, das du gerade liest.

## Nahsicht

Hier erfährst du, wie die einzelnen Bausteine aus der Nähe betrachtet aussehen.

### Perl und Mojolicious

Die Anwendung benötigt Perl 5.20.

Die Dateistruktur entspricht dem üblichen Perl-Standard:

- Das Programm liegt in »bin«.
- Die Module liegen im Verzeichnis »lib«.
- Die Tests liegen im Verzeichnis »t«.

Zusätzlich findest du im Hauptverzeichnis noch folgende Dateien:

- Die Datei »cpanfile«[@miyagawa:2017] beschreibt die Abhängigkeiten der Module. Momentan arbeiten wir nicht mit mehreren Entwicklungsstufen, daher sind hier einfach alle benötigten Module aufgeführt. Da wir auch keine besonderen Modulversionen benötigen, ist bei keinem Modul eine Version angegeben.
- Die Datei »dist.ini« ist die Konfigurationsdatei für Dist::Zilla und wird im folgenden Abschnitt erläutert.
- Die Datei ».gitlab-ci.yml« ist die Konfiguration für die »continuous integration« von Gitlab. Sie wird weiter unten näher beschrieben.
- Die Datei ».perltidyrc« enthält unsere syntaktischen Codier-Konventionen. Am sinnvollsten bindest du das Programm »perltidy« in deinen Editor ein und formatierst vor dem Check-In deine Perl-Quellen damit.
- Die Datei ».perlcriticrc« enthält unsere Programmierregeln. 

### Dist::Zilla

Dist::Zilla ist eigentlich dazu gedacht, das Leben von CPAN-Autoren zu vereinfachen. Es ist aber auch für die »normale« Entwicklung hilfreich, da dieses Werkzeug viele manuelle Arbeitsschritte erledigt. Dies ist auch dann nützlich, wenn man das Resultat *nicht* auf CPAN veröffentlichen möchte. Daher setzen wir es ein.

Zentral für Dist::Zilla ist die Konfigurationsdatei »dist.ini«. Zu Beginn dieser Datei legst du Einstellungen fest, die für die gesamte Distribution gelten. Darunter gibst du einzelne Plugins oder Bündel^[_bundle_] von Plugins an, die dann die Funktionalität erweitern.

~~~~ { .ini .numberLines }
name = postmasterfilterdebugger
version = 0.001
abstract = a debugger for the OTRS postmasterfilter
author = Gregor Goldbach <glauschwuffel@nomaden.org>
main_module = lib/OPMTools/PostmasterFilter/Debugger.pm
license = Perl_5
copyright_holder = Perl Services

[@Git]
[@Basic]

[AutoPrereqs]

[MinimumPerl]
[PkgVersion]
[CheckChangesHasContent]

[NextRelease]

[Test::ChangesHasContent]

[Test::NoTabs]
[PodCoverageTests]
[PodSyntaxTests]
[InstallGuide]
[ExecDir]
dir = scripts

[PodWeaver]
[ReadmeFromPod]

[PerlTidy]
perltidyrc = .perltidyrc
~~~~

- »name« setzt den menschenlesbaren Namen der Distribution.
- »version« legt die Versionsnummer der Distribution fest. Diese wird für alle enthaltenen Module identisch gesetzt.
- »abstract« ist eine kurze Beschreibung der Distribution. Das ist der Text, der in der _man page_ oben neben dem Namen auftaucht.
- »author« legt den Autor fest. Dieser Wert wird an allen Stellen eingesetzt, an der ein Autor angegeben wird.
- »main_module« gibt an, welches das »Hauptmodul« in der Distribution ist. Hieraus werden einige Daten automatisch ermittelt.
- »license« ist die Lizenz der Distribution. Sie bestimmt den Inhalt der Datei »LICENCE«, die automatisch angelegt wird. Je nach ausgewählter Lizenz können weitere Werte aus der Konfigurationsdatei verwendet werden. Beispiele sind in der Dokumentation der Basisklasse[@signes-software-license] aufgeführt.
- »copyright_holder« legt den Namen des Inhabers des Copyrights fest.
- »\@Git« ist ein _plugin bundle_. Es lädt eine Reihe von Plugins und belädt diese mit sinnvollen Vorgaben. Mit diesen Plugins baut Dist::Zilla im Wesentlichen ein Archiv, das den gängigen Standards entspricht und auf CPAN hochgeladen werden kann. Ich (Gregor) habe dieses Bundle aus Beispielkonfigurationen für die dist.ini übernommen. Welche Plugins enthalten sind, ist in der Dokumentation des Bundles[@signes-dist-zilla-plugin-bundle-basic] aufgeführt.
- »\@Basic« hier gilt das gleiche wie für das zuvor erwähnte Bundle.
- »AutoPrereqs« scant die Quellen und ermittelt daraus halbwegs zuverlässig die benötigten Module. Diese kannst du dann mit »dzil listdeps« ausgeben lassen und »cpanm« zum Installieren übergeben. Intern verwendet Dist::Zilla diese Abhängigkeiten, um die Dateien »Makefile.PL« und »Build.PL« zu schreiben.
- »MinimumPerl« errät aus den Quellen anhand der verwendeten Syntax, welche minimale Perl-Version benötigt wird, um das Programm laufen zu lassen. Diese Version wird dann in den Dateien »Makefile.PL« und »Build.PL« als benötigt angegeben.
- »PkgVersion« schreibt die in »dist.ini« angegebene Versionsnummer in die Module. Damit legst du effektiv _eine_ Versionsnummer für _alle_ Module fest.
- »CheckChangesHasContent« prüft vor dem Veröffentlichen der Distribution, ob in der Datei »Changes« auch Text für die Version eingetragen ist, die veröffentlicht werden soll. Damit stellst du sicher, dass du nicht versehentlich eine Version veröffentlichst, ohne zumindest eine Anmerkung für den Anwender zu hinterlassen.
- »NextVersion« ersetzt den Platzhalter »{{$NEXT}}« in der Datei »Changes« durch Datum und Versionsnummer. Dadurch stellst du einheitliche Angaben sicher und verringerst manuelle Arbeit.
- »Test::ChangesHasContent« ist die Schwester von »CheckChangesHasContent« und prüft bei »dzil test«, ob die Datei »Changes« Inhalt für die zu veröffentlichende Version enthält.
- »Test::NoTabs« prüft während »dzil test«, ob die Perl-Quellen Tabulatoren enthalten. Ich habe das eingebaut, um sicherzustellen, dass nicht mit Tabulatoren eingerückt wird.
- »PodCoverageTests« prüft, ob der Code mit POD dokumentiert ist. Mit »=for Pod::Coverage sub_name other_sub this_one_too« kannst du vorgeben, dass die angegebenen Subs dokumentiert wären.
- »PodSyntaxTests« prüft das POD auf Syntaxfehler.
- »InstallGuide« schreibt die Datei »INSTALL« und gibt dem Anwender Informationen darüber, wie er die Distribution installieren kann.
- »ExecDir« sorgt dafür, dass die Inhalte des Verzeichnisses »bin« als ausführbare Programme installiert werden, wenn du »dzil install« aufrufst.
- »PodWeaver« ist eigentlich eine Maschinerie für sich. Dieses Plugin nimmt dir die Arbeit ab, die wiederkehrenden Elemente des PODs zu schreiben. Anhand der übrigen Konfigurationswerte wird das POD in jeder deiner Dateien um beispielsweise Autor, Copyright und Lizenz ergänzt. Die langweilige Arbeit wird dir also abgenommen und du kannst dich auf den wirklich wichtigen Teil des Inhalts konzentrieren.
- »ReadmeFromPod« erstellt die Datei »README« anhand des PODs aus dem Modul, das du mit »main_module« festgelegt hast.
- »PerlTidy« jagt jede deiner Perl-Dateien durch PerlTidy, bevor die Distribution veröffentlicht wird. Dadurch stellst du sicher, dass du zumindest in dem veröffentlichten Code deine Codier-Richtlinien einhältst.

### Docker

(Was ist Docker?)

- Wir haben mit Docker nur insofern zu tun, als dass Gitlab nach jedem Commit in der CI-Stufe das Projekt in einem frischen Image neu baut
- um eine kurze Feedback-Schleife zu haben und dadurch möglichst schnell über Fehler informiert zu werden, sollte in dem Image möglichst viel von der Software installiert sein, die wir einsetzen
- aber eben auch nicht mehr, damit das image klein bleibt
- Ich (Renée) habe das Image so gebaut, dass neben Perl und den benötigten Werkzeugen auch die verwendeten Module schon installiert sind.
- Mehr zu den der Feedback-Schleife dann unter Gitlab

#### Begriffe

- man baut ein _Image_. In diesem Image ist die benötigte Software installiert. Du kannst ein lauffähiges System aus diesem Image starten.
- Es kann unterschiedliche Varianten eines Images geben. Diese Varianten werden anhand unterschiedlicher _tags_ identifiziert.
- Das gestartete Image ist dann ein _Container_.^[Das kannst du ungefähr mit _Programm_ und _Prozess_ vergleichen.]
- Das Image gibt es einmal, daraus kannst du beliebig oft einen Container erzeugen.
- Die Beschreibung, was in einem Image enthalten ist, liegt in einer Datei namens _Dockerfile_.
- _Dockerhub_ stellt Images bereit. Hier kannst du dir ein Konto anlegen und dein gebautes Image ablegen.
- Gitlab zieht sich in der CI-Stufe von Dockerhub das benötigte Image und baut dann darin die Projekte.

#### Unsere Praktiken

Dieser Abschnitt enthält unsere Praktiken, die wir uns aus der praktischen Arbeit abgeleitet haben. (Da wir noch ganz am Anfang stehen, ist dies noch nicht besonders viel.)
- Die Feedback-Schleife in Gitlab soll kurz sein. Ich möchte nach meinem Commit schnell erfahren, ob ich etwas zerstört habe. Idealerweise bekomme ich innerhalb von 10 Minuten Rückmeldung.
- Damit das möglich ist, muss in dem Image viel Software installiert sein und die Tests müssen performant sein.
- Ersteres ist die Baustelle von Docker. Das Dockerfile wird lang werden, ebenso die Bauzeit für das Image.
- Bei jedem Build auf Gitlab wartest du ein paar Minuten. Das ist auch der Fall, wenn du die ».gitlab-ci.yml« geändert hast und ihre Auswirkungen prüfen möchtest. Es ist aufgrund der anfallenden Wartezeiten daher unpraktikabel, Änderungen am Build-Prozess durch Änderungen an der ».gitlab-ci.yml« auszuprobieren. Besser ist es, wenn du das Docker-Image lokal startest, eine Bash darin öffnest, und diese Änderungen dort ausprobierst.
- Für die Pflege des Docker-Images haben wir ein Gitlab-Projekt[@goldbach-perldevenv-gitlab].
- Das fertige Image schiebt Gregor nach dem Bauen hoch zu Dockerhub[@goldbach-perldevenv-dockerhub].
- Bevor du das Image neu baust, kopierst du die Datei »cpanfile« von Hand aus dem postmasterfilterdebugger-Repo in das perldevenv-Repo.

#### Unser Dockerfile

Hier zeige ich dir unser Dockerfile und beschreibe, warum es so aufgebaut ist.

~~~~ { .ini .numberLines }
FROM ubuntu:rolling
~~~~
Hiermit gebe ich an, dass dieses Image auf dem Image »ubuntu«^[https://hub.docker.com/_/ubuntu/] mit dem Tag »rolling« basiert. Es wird aus den offiziellen Ubuntu-Quellen gebaut. »rolling« ist das aktuelle Release. Ich habe dieses Tag gewählt, weil mir dies ein annehmbarer Kompromiss zwischen Aktualität und Stabilität der installierten Software zu sein scheint.

~~~~ { .ini .numberLines startFrom=2 }
MAINTAINER = Gregor Goldbach <glauschwuffel@nomaden.org>
~~~~
»MAINTAINER« gibt denjenigen an, der dieses Image wartet und pflegt.

~~~~ { .ini .numberLines startFrom=3 }
ENV DEBIAN_FRONTEND noninteractive
~~~~
Wir wollen einen vollautomatisierten Prozess zum Erstellen des Images umsetzen. Hierbei sind Nachfragen bei der Installation von Softwarepaketen hinderlich.
Durch »ENV DEBIAN\_FRONTEND noninteractive« setzen wir die Umgebungsvariable »DEBIAN\_FRONTEND« auf den Wert, der Nachfragen bei der Installation unterdrückt.^[http://www.microhowto.info/howto/perform_an_unattended_installation_of_a_debian_package.html]

Mit »RUN« führen wir in dem System, das mit dem Image bereitgestellt wird, einen Befehl aus.

~~~~ { .ini .numberLines startFrom=5 }
RUN apt-get update
~~~~
Der hier ausgeführte Befehl aktualisiert zunächst Ubuntus Paketindizes, die Installationsquellen und sämtliche installierte Software.

Unser Image basiert auf dem letzen aktuellen Ubuntu-Release (Tag »rolling«). Wenn seit Release des Images nicht allzu viel Zeit vergangen ist, dann wird dieser Schritt kurz sein. Wenn das Release schon etwas länger her ist, dann sind wahrscheinlich mittlerweile viele Softwarepakete aktualisiert worden und hier wird entsprechend viel Software aktualisiert.

~~~~ { .ini .numberLines startFrom=6 }
RUN apt-get install -y build-essential
RUN apt-get install -y cpanminus
RUN apt-get install -y libdist-zilla-perl
RUN apt-get install -y libdist-zilla-plugin-podweaver-perl
RUN apt-get install -y libdist-zilla-plugin-test-notabs-perl
RUN apt-get install -y libdist-zilla-plugin-test-perl-critic-perl
RUN apt-get install -y libtest-bdd-cucumber-perl
RUN apt-get install -y libtest-spec-perl
RUN apt-get install -y libppi-perl
RUN apt-get install -y libppix-regexp-perl
RUN apt-get install -y libmoosex-types-path-tiny-perl
RUN apt-get install -y libdist-zilla-plugin-git-perl
RUN apt-get install -y unzip
~~~~
Danach werden Basispakete zum Entwickeln von Software, Perl und eine Reihe von Perl-Paketen installiert. Wir füllen hier unseren Perl-Werkzeugkasten und installieren »cpanminus«, »Dist::Zilla« und die Testmodule »Test::Spec« und »Test::BDD::Cucumber«. Danach installieren wir noch einige unterstützende Module.

~~~~ { .ini .numberLines startFrom=18 }
RUN apt-get install -y texlive-full
RUN apt-get install -y pandoc
~~~~
Damit wir diese Dokumentation, die du gerade liest, vollautomatisch erstellen können, installieren wir eine komplette TeX-Distribution^[https://www.tug.org/texlive/] und Pandoc^[https://pandoc.org/]. Das ermöglicht uns, die Dokumentation im Pandoc-Dialekt von Markdown^[https://pandoc.org/MANUAL.html#pandocs-markdown] zu schreiben und ein ansehnliches PDF (oder andere Formate) zu erhalten.

~~~~ { .ini .numberLines startFrom=20 }
ADD ./cpanfile cpanfile
RUN cpanm --installdeps .
~~~~
»ADD ./cpanfile cpanfile« fügt die Datei »cpanfile« dem Image hinzu. Nach Ausführung dieses Befehls wird die Datei unter dem angegeben Pfad im Image verfügbar sein. Das machen wir, da wir mit dem folgenden »RUN cpanm --installdeps .« die noch fehlenden Perl-Distributionen nachinstallieren können.

#### Zusammenfassung

Wir installieren zunächst die Softwarepakete, die als Ubuntu-Pakete vorliegen, und danach noch fehlende Perl-Distributionen direkt von CPAN.

Die Datei »cpanfile« ist eine Kopie der Datei aus dem postmasterfilterdebugger-Repository. Sie muss von Hand kopiert werden, da mir dafür noch kein sinnvoller Automatismus eingefallen ist.

Das resultierende Image ist etwa 5GB groß. Die Größe kommt im wesentlichen durch TeX-Live zustande. Ohne TeX-Live und pandoc ist das Image kleiner als 1GB.

Das Bauen des Images wird je nach Rechnerleistung mehr als eine Stunde dauern. Docker ist zwar schlau und baut das Image nicht jedesmal komplett neu, sondern durch Magie differentiell, aber beim ersten Bauen muss man richtig lange warten. Dafür erhältst du dann aber ein Image, in dem die benötigte Software bereits installiert ist.

Nach Abschluss des Bauens schiebt Gregor dann das Image auf Dockerhub hoch. Gitlab zieht sich dann nach dem nächsten Commit im postmasterfilterdebugger-Repo das aktualisierte Image. Beim ersten Mal dauert dies, ab dem zweiten Mal geht dies schneller – offensichtlich holt Gitlab nur bei neuen Versionen das Image erneut.

### Kurzreferenz

In diesem Abschnitt führe ich die Befehle auf, die ich bisher bei der Arbeit mit Docker tatsächlich eingesetzt habe.

- Ein Image lokal im Verzeichnis bauen: »docker build .«. Damit du es hinterher hochladen kannst, muss es »getaggt« sein: »docker build -t glauschwuffel/perldevenv:latest .«
- Ein Image als Container starten: »docker run -it _IMAGE_  bash«, also bei uns »docker run -it glauschwuffel/perldevenv bash«. Nach der erfolgreichen Starten des Images wird die ID des gestarteten Containers in die Konsole geschrieben.
- Container auflisten: »docker ps«. Du wirst sehr wahrscheinlich eine sehr kleine Schriftgröße eingestellt haben müssen, damit die ASCII-Tabelle in deiner Shell lesbar ist.
- Bash in dem Container ausführen: »docker exec -it _ID_ bash«. Damit logge ich mich dann efektiv als root auf dem System ein und kann darin arbeiten.
- Image auf DockerHub hochladen: »https://docs.docker.com/docker-cloud/builds/push-images/»

### Gitlab

#### Gitlab-CI

Wir verwenden ein Image, das Gregor gebaut hat. Das Image wohnt dort: <https://gitlab.com/glauschwuffel/perldevenv>. Es ist groß, weil es ein vollständiges TeXLive enthält. Das brauchen wir, damit wir diese schicke Dokumentation bauen können.

Durch Verwendung des Images erhältst du nach dem Commit innerhalb weniger Minuten Rückmeldung, wenn dein Commit etwas kaputt gemacht hat.

Ziel ist es, binnen 10 Minuten Rückmeldung zu erhalten. Gelegentlich kann es vorkommen, dass Gitlab viel zu tun hat und die Rückmeldung länger dauert. Das könnten wir vermutlich dadurch beheben, dass wir Gitlab Geld geben. Momentan (Stand September 2017) setzen wir die kostenlose Variante von Gitlab ein.

##### Artefakte zwischen Projekten austauschen

Problemstellung: Wir haben Code-Abhängigkeiten zwischen unseren Repos. Da unser Code nicht öffentlich ist, können wir aber nicht die erzeugen Archive auf CPAN veröffentlichen und von dort installieren. Wie kommen die benötigten Module von einem Repo möglichst aufwandsarm in ein anderes?

Mit Bordmitteln von Gitlab scheint es nicht so einfach zu gehen. Gregor hat zwar in v4 des APIs [Trigger](https://docs.gitlab.com/ee/ci/triggers/) gefunden, den Austausch damit aber nicht geschafft. Es ist aber möglich, über ein Token den Zugriff auf das API zu ermöglichen.

Dafür hat Gregor in seinem Profil ein »access token« generiert. Dieses müssen wir geheim halten, denn damit darf man »als Gregor« unter seinem Profil das API ansprechen! <https://gitlab.com/profile/personal_access_tokens>

In dem Projekt, das ein Artefakt aus einem anderen Projekt laden möchte, definiert du dann dieses Token als »geheime Variable« (das ist ein Begriff von Gitlab). Diese Variable ist dann als Umgebungsvariable in der .gitlab-ci.yml verfügbar: <https://gitlab.com/perlservices/otrstoolbox/settings/ci_cd>

![Einstellungen der CI in Gitlab](img/gitlab-settings-ci.png "Gitlab Settings CI"){ width=100% }

In der CI kannst du dann mit curl die Artefakte runterladen:

    "curl --header \"Private-Token: ${ARTIFACT_TOKEN}\" \
      https://gitlab.com/perlservices/otrstoolbox_common/-/jobs/artifacts\
      /master/download?job=build >archive.zip"

Achtung: Die Anführungszeichen um die Befehlszeile sind wichtig! Wenn du sie weglässt, gibt es YAML-Fehlermeldungen, die nicht hilfreich sind.

##### Vorteile dieses Vorgehens

Git bietet zwar die Möglichkeit, mit Sub-Modulen und Sub-Trees Code von anderen Repos einzubinden. Hier muss man aber einerseits genau wissen, was man tut, und kann sich andererseits auch nicht auf die Perl-Toolchain verlassen. Wenn ich also wie oben beschrieben Artefakte verwende, habe ich folgende Vorteile:

- Ich habe Artefakte von Projekten. Dadurch kann mich darauf verlassen, dass ich erfolgreich getestete und gebaute Archive installieren kann. Daraus folgt, dass das benötigte Modul gebaut werden konnte und alle Tests laufen.
- Ich kann den Perl-Werkzeugkasten zur Installation nehmen. Die Installation der benötigten Distribution wird somit laufend mitgetestet.
- Ich muss mir nicht irgendetwas aus den Quellen zusammensuchen (auch wenn es cpanm gibt ...).

