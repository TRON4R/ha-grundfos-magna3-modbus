# Changelog

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
