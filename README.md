# Dunstabzug – intelligente Koch-Erkennung

Home Assistant Blueprint zur automatischen Steuerung eines Dunstabzugs. Die Koch-Erkennung ist speziell für einen Gasherd ausgelegt, bei dem eine elektrische Leistungsaufnahme kaum als Signal verwendbar ist.

## Erkennung

Die Automation berechnet einen Koch-Score aus vier Signalen:

| Signal | Standard | Punkte |
|---|---:|---:|
| Luftfeuchtigkeit steigt deutlich | >= 0,15 %/min | +2 |
| Luftfeuchtigkeit ist erhöht | >= 45 % | +1 |
| Temperatur steigt deutlich | >= 0,08 °C/min | +1 |
| Temperatur ist erhöht | >= 24 °C | +1 |

Standardmäßig wird der Dunstabzug ab **3 Punkten** eingeschaltet.

Dadurch reicht zum Beispiel ein deutlicher Feuchteanstieg alleine noch nicht aus. Ein deutlicher Feuchteanstieg plus erhöhte Luftfeuchtigkeit oder Temperaturanstieg dagegen schon.

Die Leistungsaufnahme des Gasherds wird bewusst nicht mehr als primäres Signal verwendet.

## Nachlauf

Sobald die Kochaktivität nicht mehr erkannt wird, startet ein konfigurierbarer Nachlauf. Standardmäßig läuft der Dunstabzug noch **7 Minuten** weiter.

Während dieser Zeit wird die Automation bei neuen Sensoränderungen erneut bewertet. Beginnt die Kochaktivität wieder, wird der Nachlauf dadurch abgebrochen und der Dunstabzug bleibt an.

## Benötigte Sensoren

Es werden vier Sensoren benötigt:

1. Luftfeuchtigkeit Küche
2. Derivative-Sensor der Luftfeuchtigkeit in %/min
3. Temperatur Küche
4. Derivative-Sensor der Temperatur in °C/min

Zusätzlich werden ein Anwesenheitssensor der Küche und der zu steuernde Switch bzw. Fan benötigt.

### Beispiel für Derivative-Sensoren

```yaml
sensor:
  - platform: derivative
    source: sensor.kuche_luftfeuchtigkeit
    name: Küche Luftfeuchtigkeit Änderungsrate
    unit_time: min
    time_window: '00:05:00'

  - platform: derivative
    source: sensor.kuche_temperatur
    name: Küche Temperatur Änderungsrate
    unit_time: min
    time_window: '00:05:00'
```

Das `time_window` sorgt dafür, dass kurze Messspitzen nicht sofort als starke Änderung interpretiert werden.

## Blueprint installieren

Raw-URL:

`https://raw.githubusercontent.com/sanftleben/dunstabzug-automatisierung/main/blueprints/automation/sanftleben/dunstabzug_automatisierung.yaml`

Danach in Home Assistant unter **Einstellungen → Automatisierungen & Szenen → Blueprints** eine neue Automation aus dem Blueprint erstellen.

## Abstimmung

Die Standardwerte sind bewusst nur ein Startpunkt. Besonders die Änderungsraten hängen stark vom Aufstellungsort des Sensors und dessen Messintervall ab.

Für einen Küchen-Sensor, der beim Kochen schnell auf Feuchte reagiert, ist die Luftfeuchte typischerweise das stärkste Signal. Die Temperatur dient vor allem als zusätzliche Bestätigung.
