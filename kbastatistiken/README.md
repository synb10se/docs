# KBA-Statistiken

Dieses Projekt lädt die Neuzulassungsdaten nach Modellreihen direkt aus dem Datendienst des KBA-Statistikportals und erzeugt daraus eine HTML-Statistik, die im Browser als Vorschau angezeigt und optional dauerhaft gespeichert werden kann.

Die eigentliche Weboberfläche liegt in `index.php`. Sie lädt die Daten über `kba-arcgis.php`, startet das Python-Skript `csv2html.py` mit temporären Dateien und übernimmt die Speicherlogik.
Sowohl die Weboberfläche als auch die erzeugte Statistik enthalten einen Link zum [Impressum](https://synvoll.de/kontakt/impressum/).

## Überblick

Der typische Ablauf ist:

1. In `index.php` auf `Statistik erzeugen` klicken.
2. `kba-arcgis.php` lädt den aktuell verfügbaren Datenstand vom ArcGIS-Datendienst des KBA und schreibt ihn als temporäre CSV.
3. `csv2html.py` erzeugt daraus eine HTML-Auswertung.
4. Die Ausgabe wird im Browser als Vorschau in einem `iframe` angezeigt.
5. Erst nach Passwortprüfung und Bestätigung wird die Datei `kba-statistiken.html` aktualisiert.

![/doc/GitHub-Worflow.jpg](https://github.com/synb10se/lpmff/blob/main/kbastatistiken/doc/GitHub-Worflow.jpg)

## Dateien

### `index.php`

`index.php` ist die interaktive PHP-Oberfläche. Ihre tatsächlichen Funktionen sind:

- per Klick auf `Statistik erzeugen` die aktuellen Daten über `kba-arcgis.php` abrufen
- mit `exec()` das Python-Skript ausführen
- die erzeugte HTML-Ausgabe als Vorschau rendern
- ein `download`-Link für das aktuelle, noch nicht gespeicherte Ergebnis anbieten
- eine Vorschau nur nach erfolgreicher Verarbeitung anzeigen
- die gespeicherte Ausgabe nur nach Eingabe des korrekten Passworts überschreiben
- bei Bedarf den Zustand der Sitzung zurücksetzen

Wichtige Details:

- Es wird immer ein temporäres Input- und Outputfile für Python verwendet.
- Nach dem Erzeugen werden Anzahl der Datensätze und der Datenstand (neuester Berichtsmonat) angezeigt.
- Die Erzeugung ist ohne Passwort möglich. Jeder kann so den aktuell verfügbaren Stand abrufen und über `HTML-Datei herunterladen` exportieren. Nur das Überschreiben von `kba-statistiken.html` ist passwortgeschützt.
- Die erzeugte HTML wird in der Session als `pending_html` gespeichert.
- Die gespeicherte Datei `kba-statistiken.html` wird ausschließlich ersetzt, wenn `action=save` mit gültigem Passwort bestätigt wird. Das Erzeugen einer Vorschau oder ein fehlgeschlagener Abruf verändert die gespeicherte Datei nicht.
- Wenn kein aktuelles, ungespeichertes Ergebnis vorhanden ist, ist ein Download per `?action=download` nicht möglich.

### `kba-arcgis.php`

`kba-arcgis.php` enthält die Funktionen für den Datenabruf und wird von `index.php` eingebunden. Die Daten werden nicht über den Download-Link des Statistikportals geladen (der sich regelmäßig ändert), sondern direkt über den dahinterliegenden ArcGIS-FeatureServer. Dessen Adresse ist fest:

```
https://services-eu1.arcgis.com/U09msXRZoxesNntH/arcgis/rest/services/SP_Modellreihen/FeatureServer/0
```

Ablauf in `kba_fetch_csv()`:

- Feldnamen und Spaltenköpfe vom FeatureServer abfragen
- Anzahl der Datensätze und den neuesten Berichtsmonat ermitteln
- alle Datensätze seitenweise abrufen (max. 1000 pro Abfrage, 6 Abfragen parallel)
- daraus eine CSV im gleichen Format wie der manuelle Export schreiben (gleiche Spaltenköpfe, UTF-8 mit BOM)
- prüfen, ob alle Datensätze vollständig angekommen sind

Bei Fehlern (Dienst nicht erreichbar, unvollständige Daten, geänderte Struktur) wird eine Fehlermeldung in `index.php` angezeigt; die gespeicherte `kba-statistiken.html` bleibt unverändert.

Ein Abruf dauert etwa 10 bis 15 Sekunden.

### `csv2html.py`

Das Python-Skript erzeugt aus der CSV die HTML-Statistik. Es verwendet nur die Python-Standardbibliothek und kann auch direkt über die Kommandozeile genutzt werden:

```bash
python3 csv2html.py --input daten.csv --output kba-statistiken.html --quiet
```

Ohne Optionen verwendet das Skript `SP_Modellreihen.csv` als Eingabe, `lm-kba-statistik.html` als Ausgabe und die Modellreihen `B05,B10,C10,T03`. Mit `--models B05,B10` lässt sich die Auswertung auf eine kommagetrennte Modellliste beschränken. `--verbose` zeigt zusätzliche Verarbeitungsdetails, `--quiet` unterdrückt Statusmeldungen und `--dry-run` prüft die Eingabe, ohne die HTML-Datei zu schreiben. Die CSV muss die vom KBA-Export erwarteten Spaltenüberschriften enthalten.

Die Ausgabe enthält die für dieses Projekt relevanten Tabellen, inklusive Jahres-/Monatsfilterung und einer HTML-Darstellung für die Browser-Vorschau. In der Leapmotor-Neuzulassungstabelle zeigen die `gesamt`-Spaltenköpfe zusätzlich die Summe je Modell über alle in der erzeugten Statistik enthaltenen Berichtsmonate. Die Summen sind unabhängig von der aktuellen Jahres- und Monatsfilterung und ändern sich daher beim Filtern nicht.

### `kba-statistiken.html`

`kba-statistiken.html` ist die dauerhaft gespeicherte Ausgabedatei. Sie wird nicht bei jedem Erzeugen überschrieben; sie bleibt erst dann auf dem aktuellen Stand, wenn im Formular in `index.php` das lokale Speichern bestätigt wurde.

Diese Datei kann direkt im Browser geöffnet, auf einem Webserver veröffentlicht oder im Wiki/Forum verlinkt werden. Für sie ist weder PHP noch Python erforderlich.

## Anforderungen

Für die Weboberfläche wird ein PHP-Webserver mit aktivierter `exec()`-Funktion sowie der PHP-Erweiterung cURL benötigt. Der Server muss ausgehende HTTPS-Verbindungen zu `services-eu1.arcgis.com` aufbauen können. Zusätzlich muss Python 3 verfügbar sein. Für den direkten Skriptaufruf reicht Python 3; zusätzliche Python-Pakete sind nicht erforderlich.

Die PHP-Konstante `PYTHON_BIN` ist in `index.php` auf `python3` gesetzt:

```php
const PYTHON_BIN = 'python3';
```

Wenn Python an einem anderen Pfad liegt, muss dieser hier angepasst werden, zum Beispiel:

```php
const PYTHON_BIN = '/usr/bin/python3';
```

## Passwortschutz für das Speichern

Das lokale Speichern ist nur freigegeben, wenn die Umgebungsvariable `KBA_SAVE_PASSWORD_HASH` gesetzt ist. Sie enthält den PHP-Hash des Passworts, nicht das Klartextpasswort.

Beispiel zur Erzeugung eines Hashes:

```bash
php -r "echo password_hash('GEHEIMES_PASSWORT', PASSWORD_DEFAULT), PHP_EOL;"
```

Danach den resultierenden Hash als `KBA_SAVE_PASSWORD_HASH` in der Umgebung des PHP-Servers hinterlegen.

Ohne gültigen Hash oder mit falschem Passwort bleibt die gespeicherte Datei unverändert; die neue Vorschau bleibt in der Session erhalten, bis ein erfolgreicher Speichern-Schritt erfolgt.

## Benutzung im Browser

1. `index.php` im Webbrowser aufrufen.
2. Auf `Statistik erzeugen` klicken und den Abruf abwarten.
3. Die HTML-Auswertung erscheint als Vorschau im `iframe`, darüber der Datenstand.
4. Optional über `HTML-Datei herunterladen` die Auswertung exportieren.
5. Wenn die Vorschau korrekt ist, das Passwort eingeben und auf `Lokales Speichern bestätigen` klicken.
6. Damit wird `kba-statistiken.html` aktualisiert.

Da das KBA neue Monatsdaten nicht immer pünktlich zum Monatsanfang bereitstellt, lohnt sich ein Blick auf den angezeigten Datenstand, bevor gespeichert wird.

Um die Vorschau nach einer fehlerhaften oder veralteten Ausgabe wieder auf den gespeicherten Stand zu setzen, kann über das Formular `Zurücksetzen` der aktuelle Zustand verworfen werden.

## Inhalt der Statistik

Die generierte Seite enthält üblicherweise mehrere Tabs mit KBA-Analysen, zum Beispiel:

- Neuzulassungen für Leapmotor
- Marktvergleich Elektro- und Plug-in-Hybrid
- Marktvergleich Verbrenner vs. Elektromobilität

Die Tabellen sind mit Jahres- und Monatsfiltern ausgestattet; die Auswahl gilt für alle Tabs gleichzeitig. Über `Gesamtanzeige` können die Filter zurückgesetzt werden.

Die Ausgabe ist für Browserdarstellung optimiert: Überschriften und Spaltenköpfe bleiben bei längeren Tabellen sichtbar, und auf kleineren Bildschirmen werden die Tabs vertikal angeordnet.

## Wichtige Hinweise

- `index.php` ist kein fertiges, direkt zu verlinkendes Statistik-HTML, sondern die Verwaltungsschnittstelle für die Erstellung.
- Der eigentliche, für Besucher nutzbare Output ist `kba-statistiken.html`.
- Die in `index.php` erzeugte Vorschau wird nur in der Session gehalten und ist kein dauerhaftes Dateisystem-Artifact.
- Der Ablauf ist bewusst so gestaltet, dass erst nach manuellem Review und freigebender Bestätigung die gespeicherte Statistik aktualisiert wird.