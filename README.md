# PV-berschussladen---node-red---goe-gemini

# PV-Überschussladen mit go-eCharger und Node-RED

Dieses Projekt steuert eine go-eCharger-Wallbox dynamisch auf Basis des PV-Überschusses.  
Es werden Ladeampere in Echtzeit angepasst, optional zwischen 1- und 3-phasigem Laden umgeschaltet und der **Only-PV-Modus** unterstützt.  
Der Flow berücksichtigt unterschiedliche Datenquellen (mit/ohne Akkupriorisierung) und kann bei mehreren Fahrzeugen automatisch umschalten.

---

## Funktionen

- **Dynamische Ampere-Regelung** (min 6 A, max 16 A, Änderung nur bei ≥ 1 A Differenz)
- **Phasenumschaltung** (2 min ≥ 4.000 W → 3-phasig, 5 min < 4.000 W → 1-phasig)
- **Only-PV-Modus** (Start: 2 min > 1.200 W, Stop: 10 min < 1.000 W)
- **Fahrzeugumschaltung** zwischen Ioniq5 und Spring (gegenseitig exklusiv)
- **PV-Überschuss-Berechnung** mit zwei Varianten:
  - **Überschuss+WB** (inkl. Akkuverbrauch)
  - **Überschuss+WB+Akku** (Akku wird ignoriert → Vorrang für Wallbox)
- **Ladeziel per Dropdown setzen** (`input_select.ladeziel_dropdown` → `input_number.ladeziel`)

---

## Benötigte Home Assistant Entitäten

### 1. go-eCharger-Entitäten (IDs anpassen!)
| Entität | Typ | Beschreibung |
|---------|-----|--------------|
| `binary_sensor.go_echarger_247529_car` | binary_sensor | Auto angesteckt |
| `number.go_echarger_247529_amp` | number | Ladeampere |
| `select.go_echarger_247529_psm` | select | Phasenmodus (**Auto**, **Force single phase**, **Force three phases**) |
| `button.go_echarger_247529_frc` | button | Neutral/Reset |
| `button.go_echarger_247529_frc_2` | button | Stop |
| `button.go_echarger_247529_frc_3` | button | Start |

---

### 2. PV-/Energie-Sensoren
| Entität | Typ | Beschreibung |
|---------|-----|--------------|
| `sensor.total_dc_power` | sensor | PV-Erzeugung (W) |
| `sensor.load_power` | sensor | Hausverbrauch (W) |
| `sensor.battery_charging_power` | sensor | Akku-Ladeleistung (W, nur beim Laden positiv) |
| `sensor.go_echarger_247529_nrg_12` | sensor | Wallbox-Verbrauch (W) |

---

### 3. Helfer (Input Booleans / Selects / Numbers)
| Entität | Typ | Beschreibung |
|---------|-----|--------------|
| `input_boolean.uberschussladen` | boolean | PV-Überschussladen aktiv/inaktiv |
| `input_boolean.wallbox_vorrang` | boolean | Umschalten zwischen Überschussquelle mit/ohne Akku |
| `input_boolean.only_pv` | boolean | Only-PV-Betrieb aktiv/inaktiv |
| `input_boolean.ioniq5` | boolean | Fahrzeugauswahl: Ioniq5 |
| `input_boolean.spring` | boolean | Fahrzeugauswahl: Spring |
| `input_select.uberschuss_ladeart` | select | **1-phasig**, **3-phasig**, **Auto** |
| `input_select.ladeziel_dropdown` | select | Ladeziel-Auswahl |
| `input_number.ladeziel` | number | Ladeziel (numerisch) |

---

### 4. Template-Sensoren

#### `sensor.uberschuss_wb`
```jinja
{% set pv = states('sensor.total_dc_power') | float %}
{% set haus = states('sensor.load_power') | float %}
{% set batterie = states('sensor.battery_charging_power') | float %}
{% set wallbox = states('sensor.go_echarger_247529_nrg_12') | float %}
{% set ueberschuss = pv - haus - batterie + wallbox %}
{{ ueberschuss | round(1) }}
```

#### `sensor.uberschuss_wb_akku`
```jinja
{% set pv = states('sensor.total_dc_power') | float %}
{% set haus = states('sensor.load_power') | float %}
{% set wallbox = states('sensor.go_echarger_247529_nrg_12') | float %}
{% set ueberschuss = pv - haus + wallbox %}
{{ ueberschuss | round(1) }}
```



