# PV-Ueberschussladen node-red goe-gemini

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

### 1. go-eCharger-Entitäten („Entitäten mit …247529… bitte auf eigene go‑eCharger‑IDs anpassen.“)
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

### Anpassungen vor dem Einsatz
- [ ] Eigene go-eCharger-Entity-IDs in `templates.yaml` anpassen (`sensor.go_echarger_247529_nrg_12`)
- [ ] PV-Leistungs-Sensor (`sensor.total_dc_power`) anpassen
- [ ] Hausverbrauch-Sensor (`sensor.load_power`) anpassen
- [ ] Falls vorhanden: Batterie-Ladeleistung (`sensor.battery_charging_power`) anpassen
- [ ] Ladeleistungsgrenzen und Zeitbedingungen im Node-RED-Flow anpassen (z. B. 4 000 W / 2 Minuten)
- [ ] Min/Max Ampere im Flow prüfen (6–16 A)


### Funktionstest nach Installation
1. Prüfen, ob alle Helfer und Sensoren in Home Assistant sichtbar sind.
2. `sensor.uberschuss_wb` zeigt Werte (W) an.
3. `input_boolean.uberschussladen` einschalten → Wallbox setzt auf 6 A.
4. Simulierten PV-Überschuss erzeugen → Ampere steigen in 1 A-Schritten.
5. Über 4 kW für 2 Minuten → Phasenmodus „Force three phases“.
6. Unter 4 kW für 5 Minuten → zurück auf „Force single phase“.


### Mitmachen
Fehler gefunden oder Verbesserungsvorschlag?  
➡ Bitte im [Issue-Bereich](../../issues) melden oder Pull Request erstellen.

![Helfer_uebersicht](https://github.com/user-attachments/assets/8a11a0a3-d5b8-4361-9d7a-4b0f7e549980)

<img width="2560" height="1094" alt="Node_Teil_1" src="https://github.com/user-attachments/assets/94e74816-6f06-4e8f-ac53-61606ee48f49" />
<img width="1592" height="1123" alt="Node_Teil_2" src="https://github.com/user-attachments/assets/feb19f35-3556-43c1-8b45-6d5e3d93715f" />

