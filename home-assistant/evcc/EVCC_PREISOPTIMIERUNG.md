# evcc Preisoptimierung

[English version](EVCC_PREISOPTIMIERUNG.en.md)

## Voraussetzungen

- Der evcc Planner-Tarif muss auf Strompreise eingestellt sein.
- Die Entity `sensor.synboxen_forecast_planner` (Planertarife [Tariff-API]) ist in Home Assistant zusammen mit den anderen Tariff-APIs (Solarprognose, Einspeisung und Stromtarif) aktiviert. Die Integration ruft aktivierte Tarifprognosen beim Start ab.
- Die Smart-Preisgrenzen von SynBox-01, SynBox-02 und der Hausbatterie müssen in evcc verfügbar sein.
- Die Entität `select.synboxen_battery_mode` muss aktiviert sein, damit Home Assistant die zeitlich begrenzte Ladesperre erneuern kann.

## Bedienung

Der Schalter **evcc Preisoptimierung aktiv** (`input_boolean.evcc_smart_price_optimizer`) aktiviert oder pausiert die Automation. Er befindet sich unter den Helfern. Ein neu angelegter Schalter startet aus; Home Assistant stellt danach seinen letzten Zustand bei Neustarts wieder her.

Ist der Schalter eingeschaltet, berechnet die Automation täglich um 22:00 Uhr aus den Prognosen der folgenden Stunden Preisgrenzen und ein Nachtziel:

- Wallboxen: Cutoff der günstigsten insgesamt vier Stunden. Derselbe Wert wird auf SynBox-01 und SynBox-02 gesetzt.
- Hausbatterie: Der Ziel-Ladezustand berücksichtigt den durchschnittlichen Verbrauch der letzten Nacht, die Solarprognose von 08:00 bis 22:00 und eine Mindestreserve von 20 % für den Zeitraum 22:00 bis 08:00. So wird genug nächtliche Energie eingeplant und zugleich Platz für den erwarteten PV-Ertrag gelassen. Der aktuelle Ladezustand und die Kapazität der Batterie werden berücksichtigt. Der Ladebedarf wird mit einer angenommenen Netz-Ladeleistung von 10,8 kW in die günstigsten Planner-Intervalle umgerechnet.

Der evcc-Planner-Forecast enthält bereits Aufschläge und MwSt. (Bruttopreis) und wird daher unverändert verwendet. Die Grenzwerte werden anschließend auf evccs Schrittweite von 0,005 €/kWh aufgerundet. evcc lädt im Modus „Smart“ aus dem Netz, wenn der Planner-Preis die jeweilige Grenze nicht überschreitet. Sobald das Batterie-Ziel erreicht ist, setzt Home Assistant alle 30 Sekunden den evcc-Modus „Laden sperren“, der automatisch nach 60 Sekunden ausläuft. Unterhalb des Ziels wird die Sperre nicht erneuert; um 08:00 wird eine bestehende Sperre aufgehoben.

Um 08:00 speichert Home Assistant den aus der Hausleistung berechneten Verbrauch der vergangenen zehn Stunden als Basis für die nächste Nacht. Beim ersten Lauf wird stattdessen der 24-Stunden-Mittelwert verwendet. Die angenommene Ladeleistung ist ein gemessener Spitzenwert; ist die tatsächliche Ladeleistung niedriger, kann evcc mehr günstige Intervalle benötigen. Bei fehlendem Forecast, zu wenigen Prognoseintervallen oder ungültigen Preisgrenzen bleiben die bisherigen Preisgrenzen unverändert und das Nachtziel wird auf 0 gesetzt. Das Ändern des Schalters wirkt für die nächste Berechnung um 22:00 Uhr.

## Entladesperre für Preisspitzen

Solange der Schalter **evcc Preisoptimierung aktiv** eingeschaltet ist, prüft eine weitere Automation alle 30 Sekunden, ob die Entladung der Hausbatterie gesperrt werden soll. Sie ordnet den Restvorrat oberhalb von 20 % den teuersten Intervallen der nächsten 24 Stunden zu. Als Netzbezug je Intervall gilt der mittlere Hausverbrauch (`sensor.evcc_hausleistung_24h_mittel`) abzüglich der PV-Prognose. Der niedrigste Preis dieser Intervalle ist die Preisspitzen-Schwelle.

- Die Entladung wird gesperrt (evcc-Modus `hold`), wenn die Batterie bei 50 % oder darunter mit mehr als 100 W entlädt und der aktuelle Preis um mehr als 0,02 €/kWh unter der Schwelle liegt.
- Die Sperre wird alle 30 Sekunden erneuert, weil evcc den Modus nach 60 Sekunden zurücksetzt.
- Sie wird aufgehoben, sobald der aktuelle Preis die Schwelle erreicht, der Ladestand 20 % unterschreitet oder 50 % überschreitet.
- Der Helper `input_boolean.evcc_battery_discharge_hold` merkt sich, dass die Sperre von dieser Automation stammt. Nur dann wird sie wieder aufgehoben.
- Hat die Nachtladung-Begrenzung `holdcharge` gesetzt, greift die Automation nicht ein.

Startgrenze (50 %), Reserve (20 %) und Abstand (0,02 €/kWh) stehen als Variablen am Anfang der Automation.

## Ablauf

```mermaid
flowchart TD
	A[22:00: Optimierung aktiv] --> B[Planner- und Solarprognose lesen]
	B --> C[Nachtverbrauch lesen; erster Lauf nutzt 24-h-Mittel]
	C --> D[Ziel berechnen: mindestens 20 % plus Nachtverbrauch]
	D --> E[PV-Aufnahmeraum von 08:00 bis 22:00 berücksichtigen]
	E --> F[Netz-Ladezeit mit 10,8 kW schätzen]
	F --> G[Preisgrenze aus den günstigsten Intervallen setzen]
	G --> H{Batterie-Ziel erreicht?}
	H -- Nein --> I[Smart-Netzladung zulassen]
	H -- Ja --> J[holdcharge setzen; alle 30 Sekunden erneuern]
	I --> H
	J --> H
	K[08:00] --> L[Verbrauchsmittel der letzten 10 h speichern]
	L --> M[holdcharge aufheben]
```

## Konfiguration

Der Helper ist in `configuration.yaml` definiert; die Automation liegt in `automations.yaml`. Die von ihr verwendeten Entity-IDs sind:

- `sensor.synboxen_forecast_planner`
- `number.synboxen_synbox_01_smart_cost_limit`
- `number.synboxen_synbox_02_smart_cost_limit`
- `number.synboxen_battery_grid_charge_limit`
- `sensor.synboxen_tariff_api_solar`
- `sensor.solarnet_leistung_verbrauch`
- `sensor.synboxen_battery_soc`
- `sensor.synboxen_battery_capacity`
- `select.synboxen_battery_mode`
- `input_number.evcc_night_consumption_kwh`
- `input_number.evcc_battery_target_soc`
- `sensor.synboxen_battery_power`
- `sensor.evcc_hausleistung_24h_mittel`
- `input_boolean.evcc_battery_discharge_hold`
