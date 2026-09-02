# Changelog

## v4.3.3 (2026-09-02)

**Bugfix:** `sensor.magna3_setpoint_meter` ("MAGNA3 Actual Setpoint (m)") turned `unknown` whenever one of the underlying Modbus registers (`sensor.magna3_feedback_sensor_max`, `sensor.magna3_setpoint_range_max`, etc.) dropped out intermittently.

Cause: on the error path the template rendered the **text `unavailable`** in `state`. For a numeric sensor (`device_class: distance`, `state_class: measurement`) that is an invalid value — Home Assistant rejects it (`Received invalid sensor state: unavailable … expected a number`) and sets the sensor to `unknown`.

Fix: the guard was moved out of `state` into a dedicated `availability:` block. `state` now renders only the number; when sources are missing the sensor becomes `unavailable` instead of `unknown` and the log error disappears. The conversion formula itself is unchanged.

- Affects both YAMLs (`grundfos_magna3.yaml` and `grundfos_magna3.de.yaml`).

**Bugfix 2:** `script.magna3_set_max_flow_limit` aborted with a template error when called **without** a parameter.

Cause: the fallback read `input_number.magna3_max_flow_limit` — a helper the package deliberately ships commented out — and the `| float` had no default. `states()` returns `unknown` for a non-existent entity, which an undefaulted `| float` rejects.

Fix: the value is now built in a `variables:` block (parameter, else helper, else `-1`), and a leading `condition: template` aborts cleanly at `< 0` instead of writing 0 m3/h to register 00106. Uncommenting the optional `input_number` restores the original fallback behaviour unchanged.

- Affects both YAMLs.

## v4.3.2 (2026-05-11)

German-only release — no changes to the English YAML or dashboard.

Full "Setpoint → Sollwert" consistency sweep in the DE YAML — follow-up to v4.3 / v4.3.1, which only covered the dashboard tooltips.

User-visible texts:
- `sensor.magna3_sollwert_anforderung`: display name "MAGNA3 Setpoint Request (00104)" → "MAGNA3 Sollwert-Anforderung (00104)"
- Commented-out optional sensor: "MAGNA3 Max Flow Limit Setpoint (00106)" → "MAGNA3 Max. Volumenstrom-Sollwert (00106)"
- Script alias: "MAGNA3: Zurück zu Lokal (Setpoint übernehmen)" → "… (Sollwert übernehmen)"
- Notification text after "Einstellungen sichern": "aktueller Setpoint übernommen" → "aktueller Sollwert übernommen"
- Constant-pressure tooltip in dashboard: "festem Druck-Setpoint" → "festem Druck-Sollwert"

Internal code comments (consistency only, not visible):
- 4 inline comments in `grundfos_magna3.de.yaml` converted from "Setpoint" to "Sollwert"

**Deliberately not touched** (would break entity_ids and history): the `unique_id` values (`magna3_r00104_setpoint_req` etc.) and the automation IDs (`magna3_sync_setpoint_percent` / `_meter`). These are technical identifiers, not user-visible.

## v4.3.1 (2026-05-11)

German-only release — no changes to the English YAML or dashboard.

- "HA-Steuerung starten" tooltip: changed remaining "Setpoint-Slider" to "Sollwert-Slider" (consistency follow-up to v4.3, where the other two tooltips were already converted)

## v4.3 (2026-05-11)

German-only release — no changes to the English YAML or dashboard.

- **Control mode display fully translated to German** (`sensor.magna3_regelungsart`). Previously only Proportionaldruck and brand names (AUTOADAPT, FLOWADAPT) were in German; mode 4 still rendered as "Constant Pressure" in a German dashboard. All modes 0, 1, 3, 4, 5, 7, 8, 10, 130 are now translated. Brand names (AUTOADAPT, FLOWADAPT, AUTOADAPT(CP)) unchanged
- **Operation mode display translated** (`sensor.magna3_betriebsart`): "Auto-Control" → "Auto-Regelung", "Open Loop Min" → "Mindestkennlinie", "Open Loop Max" → "Maximumkennlinie"
- **"HA-Steuerung starten" tooltip reworded** for better clarity, now mentions the setpoint sliders and buttons as the main control surface
- **"Einstellungen sichern & HA-Steuerung beenden" and "HA-Steuerung beenden" tooltips**: replaced the word "Setpoint" with "Sollwert" (consistency with the German UI terminology)

## v4.2 (2026-05-11)

- **Cleaned up YAML header**: removed the three inline CHANGELOG blocks (v2 / v3 / v3-enhanced, ~25 lines). Going forward the YAML header carries only a `Version:` stamp and a link to this CHANGELOG — single source of truth, no more drift between inline notes and repo CHANGELOG
- Fixed the outdated install instruction "Adjust IP address below (line 23)" — line number was already wrong (host is on line 46) and would drift on every change. Replaced with a robust description ("Adjust the IP address in the modbus: block below, key 'host:'")
- Added repo URL and CHANGELOG URL to the YAML header
- **README**: added `button-card` to the "Required HACS Cards" table — was overlooked in v4 when the dashboard was migrated to `custom:button-card`. The dashboard cannot render correctly without it
- Updated the dashboard screenshot (corrected version)

## v4.1 (2026-05-11)

- **CopyToLocal fix actually works now**: switched `magna3_to_local_copy` from a single atomic write (3 → 16) to the two-step variant (write 19, wait 500 ms, write 16). Real MAGNA3 firmware did not reliably see CopyToLocal as active during the same poll cycle as the bit-0 fall, so the EEPROM copy silently failed. Two-step gives the pump time to register the armed flag before the Remote→Local edge
- Control source text "Modbus/TCP" → "Home Assistant via Modbus/TCP" (clearer in the entities card)
- Removed per-click `persistent_notification.create` from 8 control scripts (Start, Stop, → Local, Proportional, Constant Pressure, AutoAdapt, FlowAdapt, Set Max Flow Limit) — the button click itself is feedback enough, the previous behavior spammed the notification drawer
- Added confirmation notification to `magna3_reset_alarm` (rarely used, side effects worth confirming)
- Kept notification in `magna3_to_local_copy` (EEPROM save is otherwise invisible)
- **Renamed automation** `magna3_watchdog_alarm` → `magna3_forced_to_local_detected` (entity_id change; old entity becomes unavailable after reload). The old name promised a Modbus watchdog but actually fired on every user-initiated local switch. Now triggers on `binary_sensor.magna3_forced_to_local` (Reg 00201 bit 12) — only fires when the pump was forced into local mode by something other than HA (display, GO app, etc.)
- New automation `magna3_warning_notification`: fires when the pump's warning bit (Reg 00201 bit 11, orange LED) goes on, shows warning code from Reg 00206
- Updated dashboard screenshot to reflect the new buttons (Constant Pressure, Reset Alarm) and renamed labels

## v4 (2026-05-10)

- New script `magna3_to_local_copy`: fixes the pump reverting to its old setpoint after returning to local control. Performs a single-write transition (Reg 00101 = 16 = Local + CopyToLocal) so that the bus-set setpoint, control mode, and on/off state are copied into the pump's EEPROM during the Remote→Local edge (per Grundfos doc 98367081, register 00101 bit 4 "CopyToLocal")
- New script `magna3_enable_constant_pressure`: activates Constant Pressure control mode (code 4)
- Dashboard rebuilt with `custom:button-card` (HACS dependency):
  - Native tooltips on all 9 buttons (Desktop hover; not supported on touch devices)
  - Native multi-line button labels
  - Reliable icon sizing via `transform: scale()` (workaround for ha-icon SVG viewBox quirks)
  - Explicit color via top-level `color: "#44739E"` to match standard HA icon blue across themes
- Renamed buttons for self-explanation after long reuse intervals:
  - Remote Start → Start HA Control
  - Remote Stop → Stop Pump (keep HA Control)
  - → Local → End HA Control
  - → Local (adopt) → Save Settings & End HA Control
- New Constant Pressure button (mode 4) in row 2
- New Reset Alarm button in row 3 (script existed but wasn't wired up to the UI)
- Math-style icons for Proportional Pressure (`mdi:function-variant`) and Constant Pressure (`mdi:approximately-equal`) — easier to distinguish than `mdi:gauge` / `mdi:gauge-low` at small sizes

## v3-enhanced3 (2026-04-08)

- Bearing Service sensor: replaced `unknown` string with native HA `availability` template
- Added Kelvin comments to temperature sensors (Grundfos delivers K, HA converts automatically)
- Made watchdog condition more robust (`| default(now())`)
- Added comment to `magna3_set_max_flow_limit` script
- Removed AI tool attribution comments from YAML

## v3-enhanced2 (2026-02-15)

- Control source (register 00225) with dynamic icon
- Flow estimation status text (register 00221)
- Drive state text (register 00208)
- Bearing service evaluation (register 00207)

## v3-enhanced (2026-02-09)

- Replaced all deprecated `service:` entries with `action:`
- Fixed and added various device classes
- Added registers 00221 to 00225
- Added bit 12 (ForcedToLocal) to template sensor for register 00201
- State class corrections, fixed YAML indentation errors

## v3 (2026-02-08)

- Added device classes to all registers (00213-00216, 00210-00211)
- Added state classes where missing
- Added register 00106 (SetMaxFlowLimit)
- Added registers 00316 (RemotePressure1), 00320 (RemoteTemp1) (commented out)
- Added PID parameters 00110-00112 (commented out with warnings)

## v2 (2026-02-08)

- Added unique IDs (magna3_rXXXXX)
- Fixed device classes
- Optimized scan intervals (30-60s instead of 5-10s)
- Numeric sorting by register address
- Binary sensors as templates (for bit evaluation)
- Improved descriptions

## v1 (2026-02-08)

- Initial Modbus TCP configuration for Grundfos MAGNA3 via CIM 500
- Basic sensors, scripts and automations
