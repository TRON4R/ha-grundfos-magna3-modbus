# Changelog

## v3-enhanced3 (2026-04-08)

- Bearing Service Sensor: `unknown`-String durch natives HA `availability`-Template ersetzt
- Kelvin-Kommentare bei Temperatursensoren ergaenzt (Grundfos liefert K, HA konvertiert automatisch)
- Watchdog-Condition robuster gemacht (`| default(now())`)
- Kommentar bei `magna3_set_max_flow_limit` Script ergaenzt

## v3-enhanced2 (2026-02-15)

- Steuerquelle (Register 00225) mit dynamischem Icon
- Durchfluss-Status Text (Register 00221)
- Drive State Text (Register 00208)
- Bearing Service Auswertung (Register 00207)

## v3-enhanced (2026-02-09)

- Alle veralteten `service:` Eintraege durch `action:` ersetzt
- Diverse Device Classes korrigiert oder hinzugefuegt
- Register 00221 bis 00225 hinzugefuegt
- Bit 12 (ForcedToLocal) in Template Sensor fuer Register 00201 ergaenzt
- State Class Korrekturen, YAML-Einrueckungsfehler beseitigt

## v3 (2026-02-08)

- Device Classes bei allen Registern ergaenzt (00213-00216, 00210-00211)
- State Classes ergaenzt wo fehlend
- Register 00106 (SetMaxFlowLimit) hinzugefuegt
- Register 00316 (RemotePressure1), 00320 (RemoteTemp1) hinzugefuegt (auskommentiert)
- PID-Parameter 00110-00112 hinzugefuegt (auskommentiert mit Warnungen)

## v2 (2026-02-08)

- Unique IDs hinzugefuegt (magna3_rXXXXX)
- Device Classes korrigiert
- Scan-Intervalle optimiert (30-60s statt 5-10s)
- Numerische Sortierung nach Register-Adresse
- Binary Sensors als Templates (fuer Bit-Auswertung)
- Bessere Beschreibungen

## v1 (2026-02-08)

- Initiale Modbus TCP Konfiguration fuer Grundfos MAGNA3 via CIM 500
- Grundlegende Sensoren, Scripts und Automationen
