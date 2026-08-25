# Dunstabzug – intelligente Koch-Erkennung

Home Assistant Blueprint zur automatischen Steuerung eines Dunstabzugs. Die Koch-Erkennung ist speziell für einen Gasherd ausgelegt, bei dem eine elektrische Leistungsaufnahme kaum als Signal verwendbar ist.

## Erkennung

Die Automation wertet ausschließlich die **Änderung** der Luftfeuchtigkeit und Temperatur aus. Absolute Werte werden bewusst nicht als Kochsignal verwendet, weil sie vom normalen Küchenklima abhängen.

| Signal | Standard | Punkte |
|---|---:|---:|
| Luftfeuchtigkeit steigt >= 0,3 %/min | +2 |
| Luftfeuchtigkeit steigt >= 0,8 %/min | +1 zusätzlich |
| Temperatur steigt >= 0,2 °C/min | +1 |
| Temperatur steigt >= 1,0 °C/min | +1 zusätzlich |

Standardmäßig wird der Dunstabzug ab **3 Punkten** eingeschaltet.

Beispiel: Ein deutlicher Feuchteanstieg (+2) zusammen mit einem Temperaturanstieg (+1) reicht zum Einschalten. Ein einzelner kleiner Temperaturanstieg reicht nicht aus.

## Nachlauf

Sobald die Kochaktivität nicht mehr erkannt wird, startet ein konfigurierbarer Nachlauf. Standardmäßig läuft der Dunstabzug noch **7 Minuten** weiter.

Während dieser Zeit werden die aktuellen Änderungsraten weiter geprüft. Entsteht erneut ein deutlicher Anstieg, bleibt der Dunstabzug an und der Nachlauf wird durch die erneute Bewertung praktisch verlängert.

## Benötigte Sensoren

Es werden zwei normale Sensoren und deren jeweilige Änderungsraten benötigt:

1. Luftfeuchtigkeit Küche
2. Derivative-Sensor der Luftfeuchtigkeit in %/min
3. Temperatur Küche
4. Derivative-Sensor der Temperatur in °C/min

Zusätzlich werden ein Anwesenheitssensor der Küche und der zu steuernde Switch bzw. Fan benötigt.

### Beispiel für geglättete Derivative-Sensoren

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

Das 5-Minuten-Fenster glättet kurze Messspitzen. Die Änderungsraten hängen trotzdem vom Messintervall und der Position des Sensors ab.

## Blueprint installieren

Raw-URL:

`https://raw.githubusercontent.com/sanftleben/dunstabzug-automatisierung/main/blueprints/automation/sanftleben/dunstabzug_automatisierung.yaml`

Danach in Home Assistant unter **Einstellungen → Automatisierungen & Szenen → Blueprints** eine neue Automation aus dem Blueprint erstellen.

## Abstimmung

Die Standardwerte sind anhand deines gezeigten Kochverlaufs gewählt: normale Luftfeuchtigkeit um etwa 71–73 % wird ignoriert, während der deutliche Feuchte- und Temperaturanstieg beim Nudelkochen als Kochaktivität erkannt werden soll.

Die Werte sind beim Erstellen der Automation konfigurierbar. Falls sich im echten Betrieb Fehlalarme zeigen, sollten zuerst die Änderungsraten und nicht absolute Temperatur-/Feuchteschwellen angepasst werden.
