# Warming 5.3.4 – lokaler Guzzle-8-Kompatibilitätspatch

Ziel: klassische TYPO3-Installationen der Reihen 13.4 und 14.3, insbesondere ab
13.4.36 bzw. 14.3.8. Kein offizielles Release des Extension-Autors.
Die Versionsnummer 5.3.4 bleibt bewusst erhalten; dieses Dokument kennzeichnet
den lokal gepatchten Stand. Keine Zusage für zukünftige TYPO3-Hauptversionen.

## Grundlage

- https://github.com/eliashaeussler/typo3-warming/issues/1211
- https://github.com/eliashaeussler/typo3-warming/pull/1195
- Geprüfter PR-Stand: 958463192e79ad48514a2f87ca01aa6347e07d64.
- Der offene PR erweitert lediglich die Composer-Versionsbedingungen.
  Im Issue bestätigt der Autor, dass die eigentliche Guzzle-8-Umstellung noch fehlt.

## Änderungen

- Beide TYPO3-Crawler konfigurieren den Handler am Client statt pro Request.
- Vorhandene Client-Konfiguration (z. B. Authentifizierung, Proxy, TLS und Timeout)
  wird übernommen. Request-Optionen behalten ihre Priorität.
- Handler-Stacks werden vor Änderungen geklont: kein Ansammeln von Middleware,
  keine Veränderung eines gemeinsam genutzten Clients zwischen Warmup-Läufen.
- Explizite Request-Handler bleiben auch bei wiederholtem Crawling erhalten.
- Eigene ClientInterface-Implementierungen werden über ihren sendAsync-Aufruf eingebunden.
- Öffentliches Client::getConfig() ersetzt den Zugriff auf private Eigenschaften.
- Fehlerauswertung in URL-Metadaten und Datenbank-Logging berücksichtigt, dass
  RequestException unter Guzzle 8 keine getResponse()-Methode mehr besitzt.
- Subrequests erzeugen unter Guzzle 8 bei vorhandener Antwort ResponseException;
  Fehlerursache und HTTP-Antwort bleiben erhalten.
- Composer-Bedingungen entsprechend PR #1195 erweitert.
- Mitgelieferte Drittbibliotheken bleiben bytegleich zum Originalpaket.
  Kein Austausch von TYPO3-Core-Bibliotheken und kein Autoloader-Override.

## Prüfung

PHP 8.4.14; reale Guzzle-Bibliotheken und mitgelieferte cache-warmup-Bibliothek:

- Guzzle 7.16.0 / Promises 2.6.0 / PSR-7 2.13.2.
- Guzzle 8.2.0 / Promises 3.0.2 / PSR-7 3.1.0 (TYPO3 14.3.8).
- Guzzle 8.3.0 / Promises 3.1.0 / PSR-7 3.2.0.
- Je 27 erfolgreiche Assertions: beide Crawler, HTTP 200/404, Netzwerkfehler,
  wiederholte Läufe, Handler-Overrides, Client-Konfiguration, Metadaten-Header,
  unveränderter Ausgangsclient, eigener ClientInterface-Adapter,
  Subrequest-Antworten/Exceptions sowie Datenbank- und Metadaten-Fehlerbehandlung.
- PHP-Syntaxprüfung aller 726 PHP-Dateien im Paket (php -l und Parser).
- Originalpaket reproduziert mit demselben Test den gemeldeten handler-Fehler.
- ZIP-Integrität und Vergleich sämtlicher Originaldateien.

Die Tests verwenden MockHandler für HTTP und kleine TYPO3-Testdoubles.
Sie sind keine vollständigen TYPO3-Funktionstests. Eine reale Installation mit
Datenbank, DI-Container, Backend, SSE, Sitemap-Locator und Scheduler steht hier
nicht zur Verfügung. PHPStan und der vollständige Upstream-CI-Lauf wurden nicht
ausgeführt. Composer-Installationen mit extern aufgelösten Abhängigkeiten sind
nicht Gegenstand dieses klassisch gebündelten Pakets.

## Installation

1. Bestehenden Ordner typo3conf/ext/warming sichern.
2. Inhalt dieses ZIPs vollständig nach typo3conf/ext/warming entpacken.
   Classes, Configuration, Resources und ext_emconf.php liegen direkt darin.
3. TYPO3-Caches einschließlich DI-/Code-Cache über die Wartungswerkzeuge leeren.
   Bei deaktivierter OPcache-Zeitstempelprüfung zusätzlich PHP/OPcache neu starten.
4. Im Backend zunächst eine einzelne Seite, anschließend eine Sitemap aufwärmen.
5. warming:cachewarmup über den bisher genutzten CLI-/Scheduler-Aufruf testen.
6. Im Log erfolgreiche URLs sowie eine gezielt ungültige URL kontrollieren.

Bei Problemen den gesicherten Ordner zurückspielen und die Caches erneut leeren.
Ein späteres offizielles Extension-Update überschreibt diesen lokalen Patch.
