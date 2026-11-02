# Buderus MX400 in Home Assistant und evcc

Diese Anleitung nutzt die lokale, lesende MX400-API für die Buderus Logaplus M WLW 186i-12 mit EMS-plus/Logamatic BC400.

## Voraussetzungen und Grenzen

- Die Bosch-Dokumentation nennt MX400 als unterstütztes Gateway und Firmware 15.00.01 oder neuer. Welche Ressourcen eine konkrete Wärmepumpe bereitstellt, hängt von Anlage und Konfiguration ab.
- Die lokale API ist derzeit schreibgeschützt. evcc kann Verbrauch und Energie anzeigen, aber über diese Verbindung weder die Wärmepumpe regeln noch SG-Ready schalten.
- Die API verwendet HTTPS mit einem laut Bosch problematischen Zertifikat. Die lokale Integration deaktiviert deshalb die Zertifikatsprüfung. Nutze sie nur in einem vertrauenswürdigen LAN und gib MX400-Zugangsdaten oder API-Token nicht weiter.

Quellen: [Bosch K 40 RF / MX400 API](https://github.com/bosch-home-comfort/api-docs/blob/main/docs/k-40-rf.md), [Bosch OpenAPI-Spezifikation](https://github.com/bosch-home-comfort/api-docs/blob/main/openapi/k-40-rf.yaml).

## Home Assistant einrichten

Die Integration `buderus_mx400_local` liegt bereits unter `custom_components`. Sie wird über den Config-Flow eingerichtet und gehört daher **nicht** als `sensor:`-Plattform in `configuration.yaml`.

1. Home Assistant neu starten, falls die Integration noch nicht in der Integrationsauswahl erscheint.
2. Unter **Einstellungen → Geräte & Dienste → Integration hinzufügen** nach **Buderus MX400 Local API** suchen.
3. Hostname oder IP-Adresse des MX400 sowie `Login` und `Pass` vom Gateway-Aufkleber eintragen. Das Passwort kann mit Bindestrichen eingegeben werden; die Integration entfernt sie.
4. Wenn die Einrichtung eine physische Bestätigung verlangt: am MX400 WLAN und Wireless gleichzeitig eine Sekunde drücken und das Formular innerhalb von fünf Minuten erneut absenden. Das Token wird lokal erstellt und von Home Assistant gespeichert.
5. Das Abfrageintervall kann auf dem Standardwert von 30 Sekunden bleiben.

Die Integration erstellt nur Sensoren für Ressourcen, die die Anlage tatsächlich liefert. Für evcc sind diese Sensoren vorgesehen:

| Zweck | Sensorname in der Integration | Verwendung |
| --- | --- | --- |
| Momentanleistung | Leistungssumme Kompressor und Heizstab | `power` in evcc, W |
| Stromverbrauch gesamt | Stromverbrauch gesamt | `energy` in evcc, kWh, Zählerstand |
| Heizungsanteil | Stromverbrauch Heizen | Optionaler HA-Sensor; nicht zusätzlich zum Gesamtverbrauch in evcc einbinden |
| Warmwasseranteil | Stromverbrauch Warmwasser | Optionaler HA-Sensor; nicht zusätzlich zum Gesamtverbrauch in evcc einbinden |

Die tatsächlichen Entity-IDs nach dem Einrichten unter **Entwicklerwerkzeuge → Zustände** nachsehen und bei Bedarf in der folgenden evcc-Konfiguration ersetzen. Namen können in Home Assistant angepasst worden sein. Falls die Leistungssumme fehlt, prüfen, ob beide Einzelsensoren **Elektrische Kompressorleistung** und **Heizstab-Leistung** verfügbar sind; die Summe wird nur angelegt, wenn beide Ressourcen vorhanden sind.

Weitere nützliche HA-Sensoren sind Vorlauf-, Rücklauf-, Außen-, Raum- und Warmwassertemperatur sowie die Modulation und die kumulierte erzeugte Wärme. evcc benötigt diese für die reine Verbrauchsanzeige nicht.

## In evcc als Heizung anzeigen

In evcc zuerst die Home-Assistant-Instanz über die evcc-Konfiguration autorisieren. Die aktuelle Home-Assistant-Anbindung verwendet dafür die Autorisierung in der evcc-Weboberfläche; keinen Langzeitzugriffstoken als `token:` in eine neue Konfiguration kopieren.

Das folgende Fragment in die bestehende `evcc.yaml` **integrieren**. `uri` muss von evcc aus erreichbar sein; `http://homeassistant:8123` ist nur ein Beispiel für Installationen, in denen dieser Hostname aufgelöst wird. Ersetze die beiden Entity-IDs durch die in Home Assistant tatsächlich angezeigten IDs.

```yaml
meters:
  - name: buderus-heizung
    type: homeassistant
    uri: http://homeassistant:8123
    power: sensor.buderus_mx400_leistungssumme_kompressor_und_heizstab
    energy: sensor.buderus_mx400_stromverbrauch_gesamt

site:
  meters:
    consumer:
      - buderus-heizung
```

Wenn `meters:` oder `site:` in deiner `evcc.yaml` schon vorhanden sind, die Einträge in diese vorhandenen Abschnitte einfügen, nicht die Abschnitte ein zweites Mal anlegen. In der evcc-Weboberfläche den Messpunkt als **Verbraucher** mit dem Anzeigenamen **Buderus Heizung** führen. Dadurch erscheinen die Werte in der Verbraucher-/Energieansicht; die Wärmepumpe wird damit nicht zu einem steuerbaren evcc-Heizungs-Loadpoint.

Zur Plausibilitätsprüfung bei laufendem Betrieb die Leistung in Home Assistant und evcc vergleichen. Bei abgeschalteter Wärmepumpe sollte die Leistung nahe 0 W liegen; der Gesamtstromverbrauch muss als monoton steigender kWh-Zähler erkannt werden. Die MX400-API kann Werte abhängig von Anlage oder Firmware auslassen.

## Alternative: getrennte Energieauswertung

Für eine getrennte Auswertung von Heizung und Warmwasser können in Home Assistant die Sensoren `Stromverbrauch Heizen` und `Stromverbrauch Warmwasser` verwendet werden. Diese beiden Energiesensoren nicht zusätzlich zum Sensor `Stromverbrauch gesamt` als Verbraucher in evcc hinzufügen, da die Summen sonst doppelt gezählt werden. Die Integration stellt derzeit keinen getrennten Momentanleistungswert für Heizung und Warmwasser bereit.