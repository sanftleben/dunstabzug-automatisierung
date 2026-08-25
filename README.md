# Dunstabzug – automatische Koch-Erkennung

Home Assistant Blueprint zur automatischen Steuerung eines Dunstabzugs anhand von Leistungsaufnahme, Luftfeuchtigkeit und Temperatur.

## Funktionsweise

Der Dunstabzug wird eingeschaltet, wenn:

- mindestens einer der konfigurierten Sensorwerte seine Einschalt-Schwelle überschreitet,
- eine Person in der Küche erkannt wird und
- die aktuelle Uhrzeit innerhalb des konfigurierten Zeitfensters liegt.

Der Dunstabzug wird ausgeschaltet, wenn alle drei Sensorwerte ihre jeweiligen Ausschalt-Schwellen unterschreiten bzw. erreichen und dieser Zustand für die konfigurierte Ausschaltverzögerung bestehen bleibt.

Durch getrennte Ein- und Ausschaltschwellen entsteht eine Hysterese. Dadurch verhindert der Blueprint ein ständiges Umschalten, wenn ein Sensorwert um einen Grenzwert schwankt.

## Blueprint installieren

In Home Assistant kann der Blueprint über die Raw-Datei importiert werden:

`https://raw.githubusercontent.com/sanftleben/dunstabzug-automatisierung/main/blueprints/automation/sanftleben/dunstabzug_automatisierung.yaml`

Alternativ kann die YAML-Datei manuell in das Blueprint-Verzeichnis von Home Assistant kopiert werden.

## Beispielwerte

Die Standardwerte orientieren sich an einer typischen Küche:

| Sensor | Einschalten | Ausschalten |
|---|---:|---:|
| Leistung | > 1 W | <= 0,5 W |
| Luftfeuchtigkeit | > 46 % | <= 44 % |
| Temperatur | > 30 °C | <= 29 °C |

Die Werte sind beim Erstellen der Automation frei konfigurierbar.
