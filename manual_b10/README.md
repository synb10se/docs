# Leapmotor B10 – Kurzanleitung

Diese statische Webanwendung ist eine inoffizielle Kurzanleitung für den
Leapmotor B10 mit dem Infotainment-System LeapOS 3.41.30. Sie soll die
Bedienung wichtiger Fahrzeug- und Systemeinstellungen erleichtern und ersetzt
nicht das originale Benutzerhandbuch.

## Inhalt

Die Artikel sind im Menü in sieben Bereiche gegliedert:

- **Einstieg:** Überblick und Warnhinweise
- **Bedienung:** Hauptbildschirm, Statusleiste und Benutzerprofile
- **Fahrzeug:** Laden und Entladen, Türen und Fenster, Licht, Schlüssel und
  Zugang sowie Sicherheit und Kinder
- **Fahren:** Fahreinstellungen, Fahrerassistenz (ADAS) und Klimaanlage
- **Infotainment:** Ton, Sprachassistent, Anzeige, Dashcam sowie System,
  Sicherheit und Updates
- **Service:** Wartung, Notfallhinweise und technische Daten
- **Rechtliches:** Impressum

Menünamen und Funktionen können je nach Ausstattung, Markt und Softwarestand
abweichen. Die Bildschirmfotos in der Anleitung zeigen LeapOS 3.41.30.

## Bedienung

Die Seite funktioniert als statische Website ohne Serverlogik oder
Build-Schritt. Über das Menü lassen sich die Artikel auswählen; die Suche kann
mit der Taste `/` geöffnet und mit den Pfeiltasten sowie Enter bedient werden.
Bilder können zur vergrößerten Ansicht angeklickt werden. Ein Hell-/Dunkelmodus
ist verfügbar und wird im Browser gespeichert.

## Dateien

- `index.html` bindet die Inhalte und Oberfläche ein.
- `assets/content/articles/` enthält die Artikel; `menu.js` bestimmt Menügruppen
  und Reihenfolge.
- `assets/app.js` implementiert Navigation, Suche, Theme und Bildansicht.
- `assets/style.css` enthält Layout und Darstellung.
- `img/` enthält die in den Artikeln verwendeten Abbildungen.

Für die Veröffentlichung genügt es, den vollständigen Inhalt dieses
Projektverzeichnisses auf einem Webserver bereitzustellen. Die Bildpfade sind
relativ zur `index.html`; daher muss die Verzeichnisstruktur erhalten bleiben.

## Hinweis

Die Anleitung ist nicht offiziell. Für die Bedienung und Sicherheit des
Fahrzeugs ist das Original-Benutzerhandbuch maßgeblich. Den verwendeten
Softwarestand und weitere Hinweise nennt die Startseite der Webanwendung.
