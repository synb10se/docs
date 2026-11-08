# Leapmotor App – Kurzanleitung

Diese statische Webanwendung ist eine inoffizielle Kurzanleitung zur Leapmotor
App für Android und iOS. Sie dokumentiert Funktionen der App und ergänzt die
Angaben des Herstellers; sie ersetzt keine offizielle Anleitung. Die
Bildschirmfotos stammen von iOS, daher kann die Android-Oberfläche abweichen.

## Inhalt

Die Artikel sind im Menü in vier Bereiche gegliedert:

- **Einstieg:** Überblick und Einrichtung des Fahrzeugs
- **Mein Fahrzeug:** Fahrzeugübersicht, Laden, Klima und Vorbereitung,
  Bluetooth-Schlüssel und Teilen sowie Energieverbrauch
- **Persönliches Zentrum:** Fahrzeugverwaltung und App-Einstellungen
- **Rechtliches:** Impressum

Die Anleitung nennt als Stand die Leapmotor App 1.18.2 (Oktober 2026). Menüs,
Funktionen und Verfügbarkeit können sich mit App-Updates ändern.

## Bedienung

Die Seite funktioniert als statische Website ohne Serverlogik oder
Build-Schritt. Über das Menü lassen sich Artikel auswählen; die Suche kann mit
der Taste `/` geöffnet und mit den Pfeiltasten sowie Enter bedient werden.
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

Die Anleitung ist nicht offiziell. Verbindlich sind die Informationen von
Leapmotor. Details zur App-Version, zum Stand der Anleitung und zu den
Herstellerhinweisen sind auch im Startartikel und im Impressum der Webanwendung
aufgeführt.
