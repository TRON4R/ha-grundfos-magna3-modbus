# Changelog

## v4.5.2 (2026-11-17)

Beide Fehler betreffen nur die ersten Sekunden nach einem Wechsel der Regelungsart; sonst schreibt die Meter-Eingabe wie bisher sofort.

**Bugfix: Meter-Eingabe in den ersten ~2 s nach einem Regelungsart-Klick nutzte den alten Bereich.** Die Wartezeit aus v4.5.1 zählte ab dem Moment, in dem HA die neue Regelungsart gelesen hat (~2 s nach dem Klick); davor war sie 0. Jetzt wartet `magna3_set_sollwert_meter`, solange „Regelungsart setzen“ läuft (vom Klick an, also schon vor dem Schreiben), die Bereiche nachgelesen werden oder der Wechsel keine 7 s her ist. Ist nach 20 s noch nicht freigegeben, wird nicht geschrieben.

**Bugfix: Zurückstellen während der Wartezeit wurde ignoriert.** Wer nach einem Regelungsart-Wechsel z. B. von 2 m auf 3 m und gleich wieder auf 2 m (= aktueller Pumpenwert) stellte, bekam trotzdem 3 m: Die Prüfung „weicht vom Pumpenwert ab“ stand in der Bedingung der Automation, und Home Assistant startet eine `restart`-Automation nur neu, wenn die Bedingung erfüllt ist. Ebenso konnte ein %-Wert, der während der Wartezeit eingegeben wurde, später vom wartenden Meter-Auftrag überschrieben werden. Jetzt bricht jede Slider-Eingabe (% und m) zuerst einen wartenden Meter-Auftrag ab und prüft erst danach, ob geschrieben werden muss.

Sonstiges:
- Kommentare und CHANGELOG v4.5.1 präzisiert: Das Nachlesen beim Alarm-Reset verringert das Risiko, ist aber keine atomare Prüfung; `queued` serialisiert nur die Vorbereitung, nicht die ganzen Start-/Stop-/Sichern-Abläufe.
- Upgrade: Paket ersetzen, Automationen und Skripte neu laden (kein Neustart, kein `modbus.reload` nötig). Die Karte ist unverändert.
- Betrifft beide YAMLs.

**Hinweis:** Mit den letzten Releases sollte sich die ganze Bedienung in Home Assistant **deutlich flüssiger anfühlen**. Änderungen erscheinen innerhalb weniger Sekunden in allen Buttons und Anzeigen und brauchen nicht mehr 10 bis 30 Sekunden wie zuvor. Auch die Fehler, die sich früher in die Einstellungen einschleichen konnten, sollten der Vergangenheit angehören. Wem das bisherige Look & Feel nicht gefallen hat, sollte dieser Version noch eine Chance geben – es ist eine deutliche Verbesserung.

## v4.5.1 (2026-11-17)

**Bugfix: Bei Mindestdrehzahl ließ sich die Pumpe nicht mehr bedienen** (seit v4.5). Das Statusregister 00201 hatte keinen Datentyp; Home Assistant liest es dann als vorzeichenbehaftete Zahl (`int16`). Ist Bit 15 „AtMinSpeed“ gesetzt, wird der Wert negativ (gemessen: −32058 bzw. −31804, zusammen rund 48 h in 10 Tagen). Die neue Prüfung `status >= 0` in `magna3_prepare_remote` hielt das für „Pumpe nicht erreichbar“: Start, Stop, Alarm-Reset und „Einstellungen sichern“ brachen mit „Wechsel nach Remote abgebrochen“ ab (ohne zu schreiben). Jetzt `data_type: uint16`. Die Statusbit-Sensoren waren nicht betroffen.

**Bugfix: Alarm-Reset konnte einen Stop wieder aufheben.** Der Reset schreibt 7 (Remote + Ein + Reset), wartet 2 s und schrieb dann immer 3 (Remote + Ein). Ein Stop in diesen 2 s wurde dadurch aufgehoben. Jetzt liest der Reset Reg 00101 vorher frisch und schreibt 3 nur, wenn dort noch seine 7 steht. Das verringert das Risiko auch gegenüber anderen Automationen, die Reg 00101 direkt schreiben; eine atomare Prüfung ist es nicht (zwischen Lesen und Schreiben bleiben wenige Millisekunden).

**Bugfix: Meter-Eingabe direkt nach einem Regelungsart-Wechsel nutzte den alten Bereich.** Die Pumpe hat je Regelungsart eigene Sollwertbereiche (gemessen: 7,5 % = 2,49 m im alten, 1,5 m im neuen Bereich); sie werden erst 5 s nach dem Wechsel nachgelesen. `magna3_set_sollwert_meter` wartet deshalb bis 7 s nach einem Wechsel, merkt sich den Wunschwert vorher und rechnet dann mit dem neuen Bereich. Danach liest es den Pumpenwert nach. Skript und „Sync Sollwert m“ laufen im Modus `restart` (letzte Eingabe gilt); der Slider-Abgleich wartet, solange das Skript läuft. Außerhalb dieser 7 s ändert sich nichts.

Sonstiges:
- Meldungen: „Einstellungen sichern“ nennt jetzt die übernommene Regelungsart und den Sollwert in % und m (frisch aus der Pumpe gelesen, also zugleich eine Kontrolle); der Alarm-Reset meldet den tatsächlichen Zustand danach (HA-Steuerung/lokal, eingeschaltet/gestoppt) statt pauschal „eingeschaltet“.
- `magna3_prepare_remote` im Modus `queued`: Bei zwei fast gleichzeitigen Klicks wird die zweite Vorbereitung nicht mehr verworfen. Das betrifft nur die Vorbereitung; die Start-/Stop-/Sichern-Abläufe selbst laufen weiterhin unabhängig.
- README: Beispiel für Handy-Benachrichtigungen zeigt nur noch die zusätzliche `notify`-Aktion (die vollständige Automation hätte die Verbesserungen aus v4.5 überschrieben). Unterschied präzisiert: Messregister mit 0xFFFF zeigen „unbekannt“, die abgeleiteten Sensoren „nicht verfügbar“.
- Kommentar zu „Einstellungen sichern“: Eine gesicherte gestoppte Pumpe ließe sich noch am Pumpendisplay oder per Grundfos GO starten, nur nicht durch die Wärmepumpe.
- Upgrade: Paket ersetzen, Automationen und Skripte neu laden und `modbus.reload` ausführen (kein Neustart nötig). Die Karte ist unverändert.
- Betrifft beide YAMLs.

**Hinweis:** Mit den letzten Releases sollte sich die ganze Bedienung in Home Assistant **deutlich flüssiger anfühlen**. Änderungen erscheinen innerhalb weniger Sekunden in allen Buttons und Anzeigen und brauchen nicht mehr 10 bis 30 Sekunden wie zuvor. Auch die Fehler, die sich früher in die Einstellungen einschleichen konnten, sollten der Vergangenheit angehören. Wem das bisherige Look & Feel nicht gefallen hat, sollte dieser Version noch eine Chance geben – es ist eine deutliche Verbesserung.

## v4.5 (2026-11-16)

**Bugfix: „Einstellungen sichern“ im Lokalbetrieb konnte konstante Drehzahl mit 0 % ins EEPROM schreiben.** Der Button schaltete im ersten Schritt nach Remote, ohne die Vorgabe-Register vorzubelegen. Nach einem Aus/Ein der Pumpe stehen 00102/00104 auf 0; genau diese Werte wären dann gesichert worden (dieselbe Falle, gegen die v4.3.4 Remote Start absichert). Jetzt ruft auch `magna3_to_local_copy` zuerst `magna3_prepare_remote` auf, und der Button ist im Lokalbetrieb ausgegraut wie Slider und Regelungsart-Buttons.

**Bugfix: Remote-Vorbereitung konnte eine gerade gemachte Änderung zurücksetzen.** `magna3_prepare_remote` nutzte die zuletzt gelesenen Werte und schrieb sie auch, wenn die Pumpe schon in Remote war (Stop, Alarm-Reset). Wer den Sollwert änderte und innerhalb von ~2 s „Pumpe stoppen“ drückte, bekam den alten Wert zurück. Jetzt liest das Skript Status, Regelungsart, Betriebsmodus und Sollwert frisch; in Remote schreibt es nichts, im Lokalbetrieb kopiert es zusätzlich den Betriebsmodus (Reg 00204 → 00103), damit eine lokale Min-/Max-Kennlinie beim Wechsel erhalten bleibt. Werte werden auf gültige Codes bzw. 0–100 % geprüft.

**Bugfix: Fehlende Sollwert-Untergrenze wurde als 0 gerechnet.** Fiel nur Reg 00215 aus, zeigte „Sollwert (Meter)“ einen falschen Wert, und `magna3_set_sollwert_meter` hätte einen falschen Prozentwert geschrieben. Jetzt muss jeder Eingang gültig sein (0 ≤ min < max ≤ 100).

**Bugfix: Zweiter Klick auf eine Regelungsart innerhalb von 2 s ging verloren** (seit v4.4.1, durch das Nachlesen). `magna3_set_regelungsart` läuft jetzt im Modus `restart`; der letzte Klick gilt. Nur Codes aus Reg 00102 werden geschrieben; `magna3_set_sollwert_prozent` schreibt nur 0–100 %.

**Verbesserung: Slider zeigen nach einem Regelungsart-Wechsel sofort den passenden Sollwert.** Die Pumpe führt je Regelungsart einen eigenen Sollwert (gemessen: FlowAdapt 7,5 %, Proportionaldruck 22 %). `magna3_set_regelungsart` liest deshalb 2 s nach dem Schreiben neben Reg 00203 auch Reg 00338 nach; vorher zeigten die Slider bis zu 15 s den Sollwert der vorherigen Regelungsart.

**Bugfix: Meter-Slider konnte veraltet bleiben.** „Beide Slider aus Pumpenwert aktualisieren“ löst jetzt auch bei Änderungen von „Sollwert (Meter)“ aus (z. B. wenn nach einem Aus/Ein ein Eingangsregister erst später zurückkommt oder sich nach einem Regelungsart-Wechsel der Bereich ändert).

**Geändert: „nicht verfügbar“ statt Scheinwerten bei Verbindungsausfall.** Bisher zeigten bei unerreichbarer Pumpe die 11 Statusbit-Sensoren „aus“, der Alarmtext „Kein Alarm“, Energiekosten und Effizienz 0 (Energiekosten fielen in der Statistik z. B. von 37,20 € auf 0 und zurück) und die Pumpenuhr 1970. Jetzt übernehmen alle abgeleiteten Template-Sensoren die Verfügbarkeit ihrer Quellen. Ausnahme: „Durchfluss L/h“ zeigt weiter 0, damit Verlaufskurven durchgehend bleiben. Außerdem melden 24 Messregister den Grundfos-Wert 0xFFFF („nicht verfügbar“) per `nan_value` als nicht verfügbar statt z. B. 655,35 K. Wer Automationen auf diese Sensoren hat: `unavailable` statt `off`/0 berücksichtigen.

**Verbesserung: Alarm- und Warnungsmeldung mit aktuellem Code.** Statusbit und Code werden getrennt gelesen; die Meldung konnte deshalb den alten Code (oft 0) enthalten. Jetzt wird der Code vor der Meldung frisch gelesen, ein Codewechsel bei aktivem Alarm aktualisiert dieselbe Meldung (`notification_id`).

Sonstiges:
- Tooltip „Einstellungen sichern“: speichert Betriebsmodus statt „On/Off-Zustand“ und schaltet die Pumpe immer ein (ein gesichertes „Aus“ ließe die Heizkreispumpe stillstehen). Tooltip und Meldung „Alarm zurücksetzen“: schaltet die Pumpe ein.
- README: Installationshinweis „Zeile 44“ ersetzt, 00308 wird alle 15 s gelesen, nur eine Sprachfassung installieren, pumpenspezifische Werte (Gauges, Meter-Slider, 5 A), Verhalten bei Verbindungsausfall, UIX-Hinweis präzisiert (Buttons grauen auch ohne UIX aus).
- Entitätslisten: veralteten „Modbus Watchdog“ durch „Pumpe wurde auf Lokal gezwungen“ ersetzt, Warnungs-Automation ergänzt.
- Upgrade: Paket ersetzen, `modbus.reload` sowie Templates, Skripte und Automationen neu laden (kein Neustart nötig); Karte neu einfügen.
- Betrifft beide YAMLs und beide Karten.

**Hinweis:** Mit den letzten Releases sollte sich die ganze Bedienung in Home Assistant **deutlich flüssiger anfühlen**. Änderungen erscheinen innerhalb weniger Sekunden in allen Buttons und Anzeigen und brauchen nicht mehr 10 bis 30 Sekunden wie zuvor. Auch die Fehler, die sich früher in die Einstellungen einschleichen konnten, sollten der Vergangenheit angehören. Wem das bisherige Look & Feel nicht gefallen hat, sollte dieser Version noch eine Chance geben – es ist eine deutliche Verbesserung.

## v4.4.1 (2026-11-16)

**Verbesserung: Diagramme und Slider folgen einer Sollwert-Änderung nach wenigen Sekunden.** Bisher sprangen die Diagramme nach 0–15 s um (Reg 00338 wird alle 15 s gelesen) und die Slider erst 20 s nach der letzten Bewegung (Ruhezeit aus v4.3.5, nötig wegen der bis zu 15 s alten Pumpenwerte). Jetzt liest „Beide Slider aus Pumpenwert aktualisieren“ 2 s nach jeder Slider-Bewegung Reg 00338 per `homeassistant.update_entity` nach; die Ruhezeit sinkt auf 4 s. Gemessen vorher: Diagramme +3 bis +11 s, %-Slider +20 bis +21 s.

**Verbesserung: Regelungsart sofort sichtbar.** `magna3_set_regelungsart` liest 2 s nach dem Schreiben Reg 00203 nach; das Badge im Kartenkopf schaltet damit nach etwa 2 s statt nach bis zu 30 s um.

Sonstiges:
- Betrifft beide YAMLs, die Karten sind unverändert.

**Hinweis:** Mit den letzten Änderungen sollte sich die ganze Bedienung in Home Assistant deutlich flüssiger anfühlen. Änderungen erscheinen innerhalb weniger Sekunden in allen Buttons und Anzeigen und brauchen nicht mehr 10 bis 30 Sekunden wie zuvor. Auch die Fehler, die sich früher in die Einstellungen einschleichen konnten, sollten der Vergangenheit angehören. Wem das bisherige Look & Feel nicht gefallen hat, sollte dieser Version noch eine Chance geben – es ist eine deutliche Verbesserung.

## v4.4 (2026-11-16)

**Geänderte Voraussetzung: UIX statt card-mod.** Die Dashboard-Karte nutzt `uix:` statt `card_mod:` (alle 11 Blöcke). card-mod wird seit Februar 2026 nicht mehr weiterentwickelt. Mit card-mod statt [UIX](https://github.com/Lint-Free-Technology/uix) funktioniert die Karte weiter, es fehlen aber Styling und das Ausgrauen (siehe unten). Die Paket-YAML ist davon nicht betroffen.

**Neu: Bedienelemente im Lokalbetrieb ausgegraut.** Solange die Pumpe nicht in Remote ist (`binary_sensor.magna3_remote_modus` aus), sind die beiden Sollwert-Slider und die vier Regelungsart-Buttons (Proportionaldruck, Konstantdruck, AutoAdapt, FlowAdapt) blass und reagieren nicht. Im Lokalbetrieb ignoriert die Pumpe die Vorgabe-Register 00102/00104, und der nächste Remote Start überschreibt sie mit den aktiven Werten (seit v4.3.4). Eine Eingabe dort hatte also nie eine Wirkung, sah aber so aus. Slider per UIX, Buttons über die eigenen `styles:` der button-card.

**Bugfix: Slider blieb auf einem Wert stehen, den die Pumpe nie bekommen hat.** Beispiel: Slider im Lokalbetrieb auf 4 m gestellt, danach HA-Steuerung gestartet → Pumpe lief mit 3,5 m weiter, der Slider zeigte dauerhaft 4 m, weil sich der Pumpenwert nicht änderte und die Pumpen-Sync deshalb nie auslöste.

Fix: „Beide Slider aus Pumpenwert aktualisieren" löst zusätzlich bei Slider-Bewegungen und beim Wechsel der Steuerquelle (Reg 00225) aus. Es gilt eine Regel: 20 s nach der letzten Slider-Bewegung zeigen beide Slider den Pumpenwert. Eigene Slider-Änderungen der Automation (gleicher Kontext wie ihr letzter Lauf) lösen keinen Neustart aus. In Remote ändert sich dadurch praktisch nichts, denn dort hat die Pumpe den eingestellten Wert nach 20 s längst; schlägt ein Schreibvorgang fehl, zeigt der Slider danach den echten Wert.

**Verbesserung: Dashboard schaltet nach Start/Ende sofort um.** Remote Start, Remote Stop, beide Varianten von „HA-Steuerung beenden“ und Alarm-Reset lesen 2 s nach dem Schreiben von Reg 00101 die Statusbits (Reg 00201) und die Steuerquelle (Reg 00225) per `homeassistant.update_entity` nach. Vorher kam die Änderung erst mit der nächsten Abfrage (alle 30 s) an, das Ausgrauen hinkte also bis zu 30 s hinterher.

**Bugfix: Button-Icons verschwanden am Handy.** Die Buttons waren fest 110 px hoch; bei schmaler Karte brauchten „Einstellungen sichern & HA-Steuerung beenden“ und „Pumpe stoppen (HA-Steuerung bleibt)“ 4–6 Zeilen und ließen dem Icon keinen Platz (gemessen bei 342 px Bildschirmbreite: 16 px bzw. 0 px). Jetzt schrumpfen bei schmaler Button-Karte (unter 130 px Innenbreite, per Container Query in `extra_styles`) Icon von 48 auf 28 px und Schrift von 14 auf 12 px; die Höhe bleibt bei 110 px. Am Desktop bleibt alles wie bisher.

Sonstiges:
- README: UIX statt card-mod in der Tabelle der benötigten HACS-Erweiterungen; neuer Absatz „Bedienung“ (wann Slider und Buttons wirken).
- Tooltip „HA-Steuerung starten“: Hinweis auf das Ausgrauen vorher.
- Buttons haben jetzt einen festen feinen Rahmen (`extra_styles`, Farbe aus dem Theme). Vorher entfernte `vertical-stack-in-card` ihn je nach Ladezeitpunkt, am Handy fehlte er meist.
- Betrifft beide YAMLs und beide Karten.

## v4.3.5 (2026-11-16)

**Bugfix: Slider sprang beim Verstellen auf einen alten Wert zurück.**

Ursache: Der Sollwert der Pumpe (Reg 00338) wird nur alle 15 s gelesen. „Beide Slider aus Pumpenwert aktualisieren" setzte die Slider 2 s nach jeder Pumpenmeldung, auch während man noch am Slider stellte. Die Meldung enthielt dann einen Wert, den man schon wieder verlassen hatte, und der Slider sprang darauf zurück. Geschrieben wurde dabei nichts (Echo-Sperre aus v4.3.4), die Pumpe hatte den richtigen Wert, aber das Einstellen war mühsam, besonders am Handy.

Fix: Die Automation läuft im Modus `restart` und setzt die Slider erst, wenn keiner der beiden Slider in den letzten 20 s bewegt wurde, und dann mit dem in diesem Moment aktuellen Pumpenwert. Eine Änderung am Pumpendisplay erscheint weiterhin nach ~2 s, direkt nach einer Slider-Bewegung spätestens nach 20 s.

Sonstiges:
- Betrifft beide YAMLs.

## v4.3.4 (2026-11-15)

**Bugfix: Remote Start + „Zurück zu Lokal (Sollwert übernehmen)" speicherte konstante Drehzahl/Frequenz.**

Ursache: Die Vorgabe-Register 00102 (Regelungsart) und 00104 (Sollwert) stehen nach jedem Aus/Ein der Pumpe auf 0. `script.magna3_start_remote` schrieb nur Reg 00101 = 3; in Remote lief die Pumpe dann mit Regelungsart 0 (Konstante Drehzahl) und 0 % Sollwert. Ein anschließendes CopyToLocal schrieb das dauerhaft ins EEPROM (lokal danach „Konstante Frequenz"). Beobachtet ab 13.11.2026: zwei Tage Open Loop statt Proportionaldruck.

Fix: Neues internes Script `script.magna3_prepare_remote`. Es kopiert die aktiven Werte (Reg 00203 Regelungsart, Reg 00338 Sollwert) in die Vorgabe-Register 00102/00104, bevor Remote Start, Remote Stop oder Alarm-Reset (der ebenfalls nach Remote schaltet) Reg 00101 setzen. Die Pumpe läuft nach dem Wechsel also genau so weiter wie vorher. Sind die Werte nicht verfügbar, bricht es ab und legt eine Benachrichtigung an. Regelungsart und Sollwert ändert man wie bisher nach dem Remote Start.

**Bugfix: `sensor.magna3_sollwert_meter` zeigte bei Open Loop Unsinn (z. B. „43 m"), und das Meter-Script schrieb 0 %.**

Ursache: Bei Konstanter Drehzahl/Frequenz meldet die Pumpe Einheit (Reg 00209 = 12, also %), Sensor-Maximum und Sollwertbereich in Prozent. Das Template prüfte die Einheit nicht, und die Register wurden nur stündlich gelesen. `script.magna3_set_sollwert_meter` begrenzte das negative Rechenergebnis auf 0 % und schrieb es.

Fix:
- Der Meter-Sensor ist nur verfügbar, wenn Reg 00209 = 2 (m) ist und eine geregelte Betriebsart läuft (Reg 00203 nicht 0/1).
- Das Meter-Script prüft dieselben Bedingungen und bricht bei einem Ergebnis außerhalb von 0–100 % ab, statt zu begrenzen.
- Reg 00209, 00210, 00211, 00215 und 00216 werden alle 5 min statt stündlich gelesen. Die neue Automation `magna3_refresh_sensor_config` liest sie zusätzlich 5 s nach jedem Wechsel der Regelungsart sofort nach (`homeassistant.update_entity`). Reg 00212–00214 (Frequenzen, Hardware-Konstanten) bleiben bei 3600 s.
- Kommentar an Reg 00209: vollständige Liste der Einheiten-Codes laut Grundfos 98367081 (0–14).

**Bugfix: Echo-Schreibvorgänge der Slider.**

Ursache: „Beide Slider aus Pumpenwert aktualisieren" setzt bei jeder Pumpenmeldung beide `input_number`; deren Sync-Automationen schrieben die Werte sofort zurück zur Pumpe – auch falsche Werte (siehe oben) oder 0 nach einem HA-Start, solange die Pumpe noch nicht verfügbar war.

Fix:
- Die Sync-Automationen schreiben nur noch, wenn der Slider vom Pumpenwert abweicht (> 0,05 % bzw. > 0,005 m) und der Pumpenwert verfügbar ist.
- Start-Sync und Pumpen-Sync setzen einen Slider nur, wenn sein Pumpenwert verfügbar ist.

Sonstiges:
- Betrifft beide YAMLs.
- Dashboard-Tooltip „HA-Steuerung starten" ergänzt.
- `entities.*.md`: neue Entitäten sowie die bisher fehlenden `script.magna3_to_local_copy` und `script.magna3_enable_constant_pressure` nachgetragen.

## v4.3.3 (2026-09-02)

**Bugfix:** `sensor.magna3_sollwert_meter` ("MAGNA3 Sollwert (Meter)") wurde `unknown`, sobald eines der zugrundeliegenden Modbus-Register (`sensor.magna3_sensor_maximum`, `sensor.magna3_sollwertbereich_max` u. a.) zeitweise aussetzte.

Ursache: Das Template gab im Fehlerfall den **Text `unavailable`** im `state` aus. Für einen numerischen Sensor (`device_class: distance`, `state_class: measurement`) ist das ein ungültiger Wert — Home Assistant verwirft ihn (`Received invalid sensor state: unavailable … expected a number`) und setzt den Sensor auf `unknown`.

Fix: Die Fallunterscheidung wurde aus dem `state` in einen separaten `availability:`-Block verschoben. Der `state` rendert jetzt nur noch die Zahl; bei fehlenden Quellen wird der Sensor regulär `unavailable` statt `unknown`, und der Log-Fehler verschwindet. Die Umrechnungsformel selbst ist unverändert.

- Betrifft beide YAMLs (`grundfos_magna3.de.yaml` und `grundfos_magna3.yaml`).

**Bugfix 2:** `script.magna3_set_max_flow_limit` brach bei einem Aufruf **ohne** Parameter mit einem Template-Fehler ab.

Ursache: Der Fallback las `input_number.magna3_max_flow_limit` — diesen Helfer liefert das Paket bewusst nur auskommentiert mit —, und das `| float` hatte keinen Default. `states()` liefert für eine nicht existierende Entität `unknown`, worauf `| float` ohne Default wirft.

Fix: Der Wert wird jetzt in einem `variables:`-Block ermittelt (Parameter, sonst Helfer, sonst `-1`), und eine vorangestellte `condition: template` bricht bei `< 0` sauber ab, statt 0 m³/h in Register 00106 zu schreiben. Wer die optionale `input_number` einkommentiert, bekommt unverändert das ursprüngliche Verhalten.

- Betrifft beide YAMLs.

## v4.3.2 (2026-05-11)

Vollständiger „Setpoint → Sollwert"-Konsistenz-Durchgang in der DE-YAML — Nachzug zu v4.3 / v4.3.1, die nur die Dashboard-Tooltips erfasst hatten.

User-sichtbare Texte:
- `sensor.magna3_sollwert_anforderung`: Anzeigename „MAGNA3 Setpoint Request (00104)" → „MAGNA3 Sollwert-Anforderung (00104)"
- Auskommentierter optionaler Sensor: „MAGNA3 Max Flow Limit Setpoint (00106)" → „MAGNA3 Max. Volumenstrom-Sollwert (00106)"
- Script-Alias: „MAGNA3: Zurück zu Lokal (Setpoint übernehmen)" → „… (Sollwert übernehmen)"
- Notification-Text beim Klick auf „Einstellungen sichern": „aktueller Setpoint übernommen" → „aktueller Sollwert übernommen"
- Tooltip Konstantdruck im Dashboard: „festem Druck-Setpoint" → „festem Druck-Sollwert"

Interne Code-Kommentare (Konsistenz, nicht sichtbar):
- 4 Inline-Kommentare in `grundfos_magna3.de.yaml` von „Setpoint" auf „Sollwert" umgestellt

**Bewusst nicht angefasst** (würde Entity-IDs und Historie brechen): die `unique_id`-Werte (`magna3_r00104_setpoint_req` etc.) und die Automation-IDs (`magna3_sync_setpoint_percent` / `_meter`). Diese sind technische Bezeichner, nicht user-sichtbar.

## v4.3.1 (2026-05-11)

- Tooltip „HA-Steuerung starten": noch verbliebenes „Setpoint-Slider" zu „Sollwert-Slider" gemacht (Konsistenz-Nachzug zu v4.3, bei der die anderen beiden Tooltips schon umgestellt wurden)

## v4.3 (2026-05-11)

- **Regelungsart-Anzeige vollständig auf Deutsch übersetzt** (`sensor.magna3_regelungsart`). Bisher waren nur Proportionaldruck und die Markennamen (AUTOADAPT, FLOWADAPT) deutsch; Mode 4 zeigte z.B. „Constant Pressure" statt „Konstantdruck". Übersetzt sind jetzt alle Modes 0, 1, 3, 4, 5, 7, 8, 10, 130. Markennamen (AUTOADAPT, FLOWADAPT, AUTOADAPT(CP)) bleiben unverändert.
- **Betriebsart-Anzeige übersetzt** (`sensor.magna3_betriebsart`): „Auto-Control" → „Auto-Regelung", „Open Loop Min" → „Mindestkennlinie", „Open Loop Max" → „Maximumkennlinie"
- **Tooltip „HA-Steuerung starten" überarbeitet**: präziser formuliert, erwähnt jetzt Sollwert-Slider und Buttons als zentrale Steuerungselemente
- **Tooltips „Einstellungen sichern & HA-Steuerung beenden" und „HA-Steuerung beenden"**: Begriff „Setpoint" durch „Sollwert" ersetzt (Konsistenz mit der deutschen UI-Terminologie)

## v4.2 (2026-05-11)

- **YAML-Header aufgeräumt**: die drei Inline-CHANGELOG-Blöcke (v2 / v3 / v3-enhanced, ~25 Zeilen) entfernt. Die YAML enthält im Header künftig nur noch eine `Version:`-Zeile und einen Link auf diese CHANGELOG — single source of truth, keine doppelte Pflege mehr
- Falsche Install-Anweisung „IP-Adresse unten anpassen (Zeile 23)" korrigiert — die Zeilennummer war ohnehin schon falsch (Host steht auf Zeile 46) und hätte bei jeder Änderung gewandert. Ersetzt durch eine robuste Beschreibung („IP-Adresse im modbus:-Block unten anpassen, Schlüssel host:")
- Repo-URL und CHANGELOG-URL zum YAML-Header hinzugefügt
- **README**: `button-card` zur Tabelle „Erforderliche HACS-Karten" hinzugefügt — war bei v4 übersehen worden, als das Dashboard auf `custom:button-card` migriert wurde. Ohne diese Karte rendert das Dashboard nicht korrekt
- Dashboard-Screenshot aktualisiert (korrigierte Variante)

## v4.1 (2026-05-11)

- **CopyToLocal-Fix funktioniert jetzt wirklich**: `magna3_to_local_copy` umgestellt vom Single-Write (3 → 16) auf die zweistufige Variante (Write 19, 500 ms Pause, Write 16). Reale MAGNA3-Firmware hat CopyToLocal nicht zuverlässig im selben Polling-Zyklus wie die Bit-0-Flanke als aktiv erkannt, dadurch ist das EEPROM-Speichern stillschweigend ausgefallen. Mit der zweistufigen Variante hat die Pumpe Zeit, das vorbereitete Flag wahrzunehmen, bevor die Remote→Local-Flanke kommt
- Steuerquellen-Text „Modbus/TCP" → „Home Assistant via Modbus/TCP" (klarer in der Entities-Karte)
- `persistent_notification.create` aus 8 Steuerungs-Scripts entfernt (Start, Stop, → Lokal, Proportionaldruck, Konstantdruck, AutoAdapt, FlowAdapt, Max-Durchfluss setzen) — der Button-Klick ist Feedback genug, vorher wurde das Notification-Drawer mit jedem Klick zugespammt
- Bestätigungs-Notification zu `magna3_reset_alarm` hinzugefügt (selten benutzt, Nebenwirkungen rechtfertigen Bestätigung)
- Notification in `magna3_to_local_copy` beibehalten (EEPROM-Speichern wäre sonst unsichtbar)
- **Automation umbenannt** `magna3_watchdog_alarm` → `magna3_forced_to_local_detected` (entity_id ändert sich; alte Entität wird nach Reload `unavailable`). Der alte Name suggerierte einen Modbus-Watchdog, feuerte aber bei jedem nutzerinitiierten Wechsel auf Lokal. Trigger jetzt `binary_sensor.magna3_zwang_auf_handbetrieb` (Reg 00201 Bit 12) — feuert nur, wenn die Pumpe von etwas anderem als HA in den Lokal-Modus gezwungen wurde (Display, GO App etc.)
- Neue Automation `magna3_warning_notification`: feuert, wenn das Warnung-Bit (Reg 00201 Bit 11, orange LED) angeht, zeigt Warnung-Code aus Reg 00206
- Dashboard-Screenshot aktualisiert (neue Buttons Konstantdruck, Alarm zurücksetzen und neue Beschriftungen)

## v4 (2026-05-10)

- Neues Script `magna3_to_local_copy`: Behebt das Zurückspringen der Pumpe auf den alten Setpoint nach Rückgabe der Steuerung an die Pumpe. Schreibt in einem einzigen Write Reg 00101 = 16 (Local + CopyToLocal), sodass beim Remote→Local-Wechsel der Bus-Setpoint, die Regelungsart und der On/Off-Zustand ins Pumpen-EEPROM kopiert werden (laut Grundfos-Dok 98367081, Reg 00101 Bit 4 „CopyToLocal")
- Neues Script `magna3_enable_constant_pressure`: aktiviert die Regelungsart Konstantdruck (Code 4)
- Dashboard neu aufgebaut mit `custom:button-card` (HACS-Abhängigkeit):
  - Native Tooltips auf allen 9 Buttons (nur Desktop-Hover; auf Touchgeräten nicht unterstützt)
  - Native mehrzeilige Button-Beschriftungen
  - Zuverlässiges Icon-Sizing via `transform: scale()` (Workaround für ha-icon SVG-viewBox-Eigenheiten)
  - Explizite Farbe via Top-Level `color: "#44739E"` für konsistentes HA-Icon-Blau über alle Themes
- Buttons umbenannt für Selbsterklärung nach längeren Nutzungspausen:
  - Remote Start → HA-Steuerung starten
  - Remote Stop → Pumpe stoppen (HA-Steuerung bleibt)
  - → Lokal → HA-Steuerung beenden
  - → Lokal (übernehmen) → Einstellungen sichern & HA-Steuerung beenden
- Neuer Konstantdruck-Button (Mode 4) in Reihe 2
- Neuer „Alarm zurücksetzen"-Button in Reihe 3 (Script war vorhanden, aber nicht im UI verlinkt)
- Mathe-Icons für Proportionaldruck (`mdi:function-variant`) und Konstantdruck (`mdi:approximately-equal`) — bei kleinen Größen leichter zu unterscheiden als `mdi:gauge` / `mdi:gauge-low`

## v3-enhanced3 (2026-04-08)

- Bearing Service Sensor: `unknown`-String durch natives HA `availability`-Template ersetzt
- Kelvin-Kommentare bei Temperatursensoren ergänzt (Grundfos liefert K, HA konvertiert automatisch)
- Watchdog-Condition robuster gemacht (`| default(now())`)
- Kommentar bei `magna3_set_max_flow_limit` Script ergänzt
- KI-Tool-Attributierungskommentare aus YAML entfernt

## v3-enhanced2 (2026-02-15)

- Steuerquelle (Register 00225) mit dynamischem Icon
- Durchfluss-Status Text (Register 00221)
- Drive State Text (Register 00208)
- Bearing Service Auswertung (Register 00207)

## v3-enhanced (2026-02-09)

- Alle veralteten `service:` Einträge durch `action:` ersetzt
- Diverse Device Classes korrigiert oder hinzugefügt
- Register 00221 bis 00225 hinzugefügt
- Bit 12 (ForcedToLocal) in Template Sensor für Register 00201 ergänzt
- State Class Korrekturen, YAML-Einrückungsfehler beseitigt

## v3 (2026-02-08)

- Device Classes bei allen Registern ergänzt (00213-00216, 00210-00211)
- State Classes ergänzt wo fehlend
- Register 00106 (SetMaxFlowLimit) hinzugefügt
- Register 00316 (RemotePressure1), 00320 (RemoteTemp1) hinzugefügt (auskommentiert)
- PID-Parameter 00110-00112 hinzugefügt (auskommentiert mit Warnungen)

## v2 (2026-02-08)

- Unique IDs hinzugefügt (magna3_rXXXXX)
- Device Classes korrigiert
- Scan-Intervalle optimiert (30-60s statt 5-10s)
- Numerische Sortierung nach Register-Adresse
- Binary Sensors als Templates (für Bit-Auswertung)
- Bessere Beschreibungen

## v1 (2026-02-08)

- Initiale Modbus TCP Konfiguration für Grundfos MAGNA3 via CIM 500
- Grundlegende Sensoren, Scripts und Automationen
