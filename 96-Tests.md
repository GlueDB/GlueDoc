# Tests

Unser Ziel ist es, möglichst viele Tests zu schreiben. Damit wollen wir eine hohe Qualität unserer Software und
unseres Codes gewährleisten. Wir unterscheiden dabei mehrere Ebenen:

* Datenbanktests
* Core-Tests
* API-Tests
* Frontendtests

Für die Tests verwenden wir Testmodule, die auf CPAN verfügbar sind, vor allem

* Test::DBIx::Class
* Test::Most
* Test::Mojo
* Test::BDD::Cucumber
* Test::PhantomJS

Alle diese Module halten sich an das Test::Anything-Protokoll (TAP). So können wir einfacher die
Ergebnisse der Tests automatisiert auswerten.

## Datenbanktests

Schon die Klassen, die für den Datenbankzugriff zuständig sind, müssen ausgiebig getestet werden, da das
die Basis für eine funktionierende API ist.

## Core-Tests

Mit Core-Tests sind Tests für alle Glue::Core::*-Module gemeint. Tests für diese Module geben einen guten
Einblick darin, ob Hilfsfunktionen richtig funktionieren und ob die einzelnen Teile gut miteinander arbeiten.

## API-Tests

## Frontendtests

