# Changelog

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
