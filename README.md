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

## Schutz gegen Fehlauslösung beim Betreten

Das blosse Betreten der Küche kann den Dunstabzug nicht starten. Anwesenheit ist reine Bedingung und niemals Auslöser. Zwei Zeitschranken müssen zusätzlich erfüllt sein:

| Zeitschranke | Standard | Bedeutung |
|---|---:|---|
| Anwesenheits-Vorlaufzeit | 5 min | So lange muss die Küche bereits durchgehend belegt sein, bevor eingeschaltet werden darf |
| Bestätigungsdauer | 3 min | So lange muss eine Änderungsrate ununterbrochen über ihrer Schwelle liegen, bevor der Score überhaupt bewertet wird |

Die Vorlaufzeit gibt den Derivative-Sensoren Zeit, sich auf die neue Raumsituation einzupendeln. Die Bestätigungsdauer filtert kurze Messspitzen heraus, wie sie durch Türöffnen, Körperwärme oder Messrauschen entstehen. Eingeschaltet wird also frühestens nach dem längeren der beiden Zeiträume.

Bei einem Gasherd mit Topf auf der Flamme ist der Feuchteanstieg lange vor Ablauf dieser Fristen stabil. Praktisch geht dadurch kaum Reaktionszeit verloren.

Falls trotzdem noch Fehlalarme auftreten, zuerst die Bestätigungsdauer erhöhen, danach die Änderungsraten anheben. Absolute Temperatur- oder Feuchteschwellen sind bewusst kein Bestandteil der Erkennung.

## Nachlauf

Der Dunstabzug schaltet aus, wenn **beide** Änderungsraten die Nachlaufzeit lang durchgehend unter ihren Koch-Schwellen liegen. Standard sind **7 Minuten**.

Entsteht während dieser Zeit erneut ein Anstieg über eine Schwelle, wird der Nachlauf verworfen und beginnt erst nach dem nächsten Abfall neu. Der Dunstabzug bleibt in diesem Fall an.

Zusätzlich prüft die späteste Einschaltzeit einmal täglich, ob abgeschaltet werden kann. Das dient als Sicherheitsnetz, falls die Automation neu geladen wurde, während der Dunstabzug lief.

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

## Verifikation im Betrieb

Die Automation lässt sich nur im laufenden Home Assistant sinnvoll prüfen. Unter **Einstellungen → Automatisierungen → Dunstabzug → Traces**:

1. **Betreten ohne Kochen** darf gar keinen Trace erzeugen. Erscheint trotzdem einer, ist noch ein Anwesenheits-Trigger aktiv.
2. **Kochbeginn** erzeugt einen Trace mit dem Zweig *Dunstabzug einschalten*. Im Trace sichtbar: welcher Änderungsraten-Trigger gefeuert hat und welchen Wert `cooking_score` hatte.
3. **Kochende** erzeugt nach Ablauf der Nachlaufzeit einen Trace mit dem Zweig *Dunstabzug ausschalten*.

Ein abgebrochener Trace im Einschaltzweig ist normal und gewollt: er bedeutet, dass ein Anstieg zwar lange genug anhielt, aber Vorlaufzeit, Zeitfenster oder Score nicht erfüllt waren.

## Abstimmung

Die Standardwerte sind anhand deines gezeigten Kochverlaufs gewählt: normale Luftfeuchtigkeit um etwa 71–73 % wird ignoriert, während der deutliche Feuchte- und Temperaturanstieg beim Nudelkochen als Kochaktivität erkannt werden soll.

Alle Werte sind beim Erstellen der Automation konfigurierbar.
