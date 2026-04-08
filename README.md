<img src="images/logo.png" alt="MAGNA3 Modbus Logo" width="120" align="left" style="margin-right:16px;"/>

### Grundfos MAGNA3
**Home Assistant Modbus TCP**

<br clear="left"/>

**YAML-Package zur Integration einer Grundfos MAGNA3 Heizkreispumpe in Home Assistant via Modbus TCP (CIM 500 Modul).**

<a href="README.en.md">English version</a>

---

## Funktionsumfang

- **40+ Sensoren**: Förderhöhe, Durchfluss, Drehzahl, Leistung, Temperaturen, Energieverbrauch, Betriebsstunden, PID-Parameter u.v.m.
- **Status-Bits**: 12 Binary Sensors aus dem Status-Register (Pumpe läuft, Alarm, Warnung, Remote-Modus, ...)
- **Berechnete Werte**: Effizienz, Energiekosten, Sollwert in Meter, Software-Version (BCD-Dekodierung), Alarm-/Warnungstexte
- **Steuerung**: Remote Start/Stop, Regelungsart wechseln (AUTOADAPT, FLOWADAPT, Proportionaldruck), Sollwert setzen (% und Meter)
- **Automationen**: Watchdog, Alarm-Benachrichtigung, Strom-Warnung, bidirektionale Sollwert-Synchronisation
- **Dashboard-Karte**: Fertige Lovelace-Karte mit Gauges, Steuerbuttons und Statusanzeigen

## Voraussetzungen

- Home Assistant (beliebige aktuelle Version)
- Grundfos MAGNA3 Pumpe mit **CIM 500** Kommunikationsmodul (Modbus TCP)
- Netzwerkverbindung zwischen HA und CIM 500

## Installation

1. Datei `grundfos_magna3.yaml` nach `/config/packages/` kopieren
2. In `configuration.yaml` sicherstellen, dass Packages aktiviert sind:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
3. IP-Adresse des CIM 500 anpassen (Zeile 44 in der YAML):
   ```yaml
   host: 192.168.2.7  # ← ANPASSEN: IP-Adresse Deines CIM 500
   ```
4. Home Assistant neu starten

## Dashboard-Karte (optional)

<p align="center">
  <img src="images/dashboard_screenshot.png" alt="MAGNA3 Dashboard" width="400"/>
</p>

Die Datei `dashboard_card.yaml` enthält eine fertige Lovelace-Karte mit Gauges, Steuerbuttons und Statusanzeigen.

**Verwendung:** Inhalt von `dashboard_card.yaml` im Dashboard-Editor als manuelle Karte (YAML) einfügen.

### Erforderliche HACS-Karten

Die Dashboard-Karte benötigt folgende HACS Frontend-Erweiterungen:

| HACS-Karte | Zweck | HACS-Suche |
|---|---|---|
| [**Vertical Stack In Card**](https://github.com/ofekashery/vertical-stack-in-card) | Äußerer Container ohne Rahmen | `vertical-stack-in-card` |
| [**card-mod**](https://github.com/thomasloven/lovelace-card-mod) | CSS-Styling (Rahmen entfernen, Farben) | `card-mod` |
| [**Mushroom**](https://github.com/piitaya/lovelace-mushroom) | Template-Karte für Durchfluss-Status | `mushroom` |

Ohne diese Karten wird die Dashboard-Karte nicht korrekt dargestellt. Die Modbus-YAML selbst funktioniert unabhängig davon.

## Registerübersicht

### Messwerte (Read-Only)

| Register | Sensor | Einheit | Intervall |
|---|---|---|---|
| 00201 | Status-Bits (12 Binary Sensors) | Bitfeld | 30s |
| 00202 | Prozess-Rückmeldung | % | 30s |
| 00301 | Förderhöhe | m | 15s |
| 00302 | Durchfluss | m³/h | 15s |
| 00303 | Relative Leistung | % | 15s |
| 00304 | Drehzahl | rpm | 15s |
| 00305 | Frequenz | Hz | 15s |
| 00308 | Sollwert (aktuell) | % | 30s |
| 00309 | Motorstrom | A | 30s |
| 00310 | DC-Link Spannung | V | 30s |
| 00312-13 | Leistungsaufnahme | W | 15s |
| 00321 | Elektronik-Temperatur | K | 15s |
| 00322 | Medientemperatur | K | 15s |
| 00326 | Spez. Energieverbrauch | Wh/m³ | 300s |
| 00327-28 | Betriebsstunden (laufend) | h | 300s |
| 00329-30 | Betriebsstunden (gesamt) | h | 300s |
| 00332-33 | Energieverbrauch | kWh | 300s |
| 00334-35 | Anzahl Starts | - | 300s |
| 00338 | Sollwert (Benutzer) | % | 15s |
| 00339 | Differenzdruck | bar | 30s |
| 00357-58 | Gepumptes Volumen | m³ | 300s |

### Steuerregister (Read-Write)

| Register | Funktion | Werte |
|---|---|---|
| 00101 | Control Register | Bit 0: Remote, Bit 1: On/Off, Bit 2: Reset |
| 00102 | Regelungsart | 6=Proportional, 128=AUTOADAPT, 129=FLOWADAPT |
| 00104 | Sollwert | 0-10000 (= 0-100%) |

### Optionale Register (auskommentiert)

Die YAML enthält weitere Register als auskommentierte Blöcke:
- **00105** RelayControl (nicht relevant für MAGNA3)
- **00110-00112** PID-Parameter (Kp, Ti, Regelrichtung) — **nur für Experten!**
- **00316** Remote-Drucksensor
- **00320** Remote-Temperatursensor
- **00352-00356** Wärmeenergie (bei externem Temperatursensor)

## Anpassungen

### Strompreis

In der YAML den Wert `0.30` für die Energiekostenberechnung anpassen:
```yaml
state: >
  {{ (states('sensor.magna3_energieverbrauch') | float(0) * 0.30) | round(2) }}
  # ↑ Strompreis anpassen!
```

### Scan-Intervalle

Die Intervalle sind konservativ gewählt (15-300s). Bei Bedarf können sie angepasst werden. Zu aggressive Intervalle (< 5s) können die Modbus-Kommunikation belasten.

## Dokumentation

Basiert auf dem offiziellen Grundfos Modbus-Dokument:
**98367081 05.2025 — Modbus for Grundfos pumps**

## Lizenz

[MIT](LICENSE)
