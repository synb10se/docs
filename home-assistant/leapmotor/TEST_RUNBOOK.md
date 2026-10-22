# Leapmotor HA Test Runbook

Stand: 2026-04-27

## Ziel

Die Integration in Home Assistant zuerst sicher importieren und nur danach
gezielt einzelne echte Fahrzeugfunktionen pruefen.

## Haftungsausschluss

Diese Integration ist inoffiziell und reverse-engineered. Nutzung auf eigenes
Risiko. Es gibt keine Haftung fuer Account-Sperren, API-Aenderungen,
fehlgeschlagene Kommandos, Fahrzeugfolgen oder sonstige Auswirkungen. Echte
Fahrzeugaktionen nur bewusst und in sicherem Fahrzeugzustand testen.

## Voraussetzungen

- Home Assistant laeuft
- `curl` ist im Home-Assistant-Host oder Container verfuegbar
- `custom_components/leapmotor` liegt im HA-Konfigurationsordner
- Home Assistant wurde nach dem Kopieren neu gestartet
- Zugangsdaten fuer das richtige Leapmotor-Konto sind bekannt
- `app_cert.pem` und `app_key.pem` liegen unter `config/leapmotor/` vor oder werden im Setup
  hochgeladen/eingefuegt
- Fahrzeug-PIN ist bekannt, falls Steuerfunktionen getestet werden sollen

## Installation

1. `custom_components/leapmotor` nach `config/custom_components/leapmotor` kopieren.
2. Home Assistant neu starten.
3. Unter `Einstellungen -> Geraete & Dienste -> Integration hinzufuegen` nach `Leapmotor` suchen.
4. Benutzername, Passwort, Zertifikat, privaten Schluessel und Intervall eingeben.
5. Fahrzeug-PIN optional eintragen. Ohne PIN bleiben die meisten
   Remote-Control-Aktionen deaktiviert.

## Sicherer Ersttest

1. Nur die Integration anlegen.
2. Pruefen, ob Fahrzeug, Sensoren, Binary Sensors und Standort sauber erscheinen.
3. Noch keinen Button druecken.
4. Im Log pruefen, ob Login, Fahrzeugliste und Statusabruf fehlerfrei laufen.
5. Pruefen, ob das richtige Fahrzeug erkannt wurde:
   - VIN
   - Modell
   - Shared-Car oder Main-Account plausibel
6. Falls keine Fahrzeug-PIN eingetragen wurde: pruefen, dass Remote-Buttons
   nicht verfuegbar sind und die Verriegelung nur den Status liefert.

## Empfohlene Testreihenfolge fuer Live-Aktionen

Diese Reihenfolge minimiert Risiko und hilft beim Eingrenzen:

1. `Fahrzeug orten`
2. `Entriegeln`
3. `Verriegeln`
4. `A/C-Schalter`
5. `Schnelles Kuehlen`
6. `Schnelles Beheizen`
7. `Windschutzscheibe auftauen`
8. `Batterie vorwaermen`
9. `Sonnenblende`
10. `Kofferraum`
11. `Fensteraktion`
12. `leapmotor.send_destination` mit einem bewusst gewaehlten Ziel

## Vorsichtsregeln

- Immer nur genau eine Live-Aktion testen.
- Nach jeder Aktion auf das Ergebnis am Fahrzeug warten.
- `Kofferraum`, `Sonnenblende`, Klima, `Fensteraktion` und Zielversand
  erst testen, wenn `Fahrzeug orten`, `Entriegeln` und `Verriegeln`
  sauber funktioniert haben.
- `Fensteraktion` nur bewusst testen, weil die Semantik der
  beobachteten Fenster-Unteraktion noch nicht vollstaendig geklaert ist.
- Zielversand nur bewusst testen, weil das Ziel direkt an die Fahrzeugnavigation
  gesendet wird.

## Modellhinweis

- Der staerkste Live-Beleg liegt aktuell fuer den `C10` vor.
- `B10` und `T03` sollten architektonisch gute Chancen haben, aber einzelne
  Funktionen koennen modellabhaengig fehlen oder serverseitig abgewiesen werden.
- Bei nicht unterstuetzten Funktionen sollte Home Assistant einen klaren Fehler
  fuer genau diese Aktion zeigen, ohne die gesamte Integration zu zerlegen.

## Was bei Fehlern festhalten

- Welche Aktion gedrueckt wurde
- Fahrzeugmodell
- Main- oder Shared-Account
- exakte Fehlermeldung in Home Assistant
- relevante Logzeilen aus Home Assistant
