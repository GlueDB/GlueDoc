# Architektur

Wir setzen bei der Entwicklung den "API first"-Ansatz ein. Das heißt, dass alle Funktionen
erst in der API existieren müssen, bevor das Frontend diese Funktionalitäten nutzt.

Das Frontend ist ein eigener Service und bereitet die Anfragen auf und leitet diese an
die API weiter. Und auf dem Rückweg wird die Antwort der API ausgewertet und dann an den
Client geschickt.

TODO: Schematische Darstellung Architektur


