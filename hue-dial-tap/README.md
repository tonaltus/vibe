# Hue Dial Tap - Home Assistant Automation

Dieser Ordner enthält eine **Home Assistant Automation**, um einen **Zigbee-Handschalter mit Drehring** (z. B. Hue Dial Switch oder ähnliche Geräte) für die Steuerung einer Lampe zu nutzen.

## 📌 Zweck
- **Taste 1**: Ein-/Ausschalten (Toggle) der Lampe **Dach-Lampe-TV**
- **Drehring**: Helligkeit der Lampe stufenlos regeln
  - **Links drehen**: Dunkler
  - **Rechts drehen**: Heller

Die Automation ist so strukturiert, dass sie **nur Taste 1** nutzt, sodass die anderen Tasten (2, 3, 4) später für weitere Lampen oder Funktionen genutzt werden können.

---

## 🛠️ Voraussetzungen

### 1. **Device ID des Schalters ermitteln**
- Gehe in Home Assistant zu **Entwicklerwerkzeuge → Zustände**
- Suche nach deinem Zigbee-Schalter (z. B. `sensor.zigbee_schalter_...` oder `light.zigbee_...`)
- Notiere die **`device_id`** aus den Attributen (z. B. `0x00124b001234abcd`)

### 2. **Input Helpers für Dimm-Einstellungen (optional, aber empfohlen)**
- Gehe zu **Einstellungen → Geräte & Dienste → Helfer → + Hinzufügen**
- Erstelle zwei **Zahl-Helfer**:
  - **Name**: `Dimm Schritt`
    **Typ**: Zahl
    **Min**: 1, **Max**: 100, **Schrittweite**: 1, **Anfangswert**: `10`
  - **Name**: `Dimm Übergangszeit`
    **Typ**: Zahl
    **Min**: 0, **Max**: 5, **Schrittweite**: 0.1, **Anfangswert**: `0.5`

---

## 📜 Dateien in diesem Ordner
| Datei | Beschreibung |
|-------|--------------|
| [`automation.yaml`](automation.yaml) | Die Haupt-Automation für Taste 1 und Drehring |

---

## 🚀 Installation

1. **Automation in Home Assistant importieren**:
   - Kopiere den Inhalt von [`automation.yaml`](automation.yaml) in den **YAML-Editor** deines Home Assistant Automationseditors.
   - Ersetze **`DEIN_SCHALTER_DEVICE_ID`** durch die tatsächliche `device_id` deines Schalters.
   - Passe ggf. `endpoint`, `command` und `action` an (siehe [Event-Daten prüfen](#event-daten-prüfen)).

2. **Speichern und testen**:
   - Speichere die Automation.
   - Teste die Funktionen:
     - **Taste 1 drücken** → Lampe schaltet um
     - **Drehring links drehen** → Lampe wird dunkler
     - **Drehring rechts drehen** → Lampe wird heller

---

## ⚙️ Anpassungen

### Event-Daten prüfen (falls nötig)
Falls die Automation nicht funktioniert, prüfe die **tatsächlichen Event-Daten** deines Schalters:
1. Gehe zu **Entwicklerwerkzeuge → Ereignisse**
2. Drücke Taste 1 oder drehe am Ring
3. Suche nach Ereignissen mit `zha_event` oder `zigbee2mqtt_event`
4. Passe die `event_data` in der [`automation.yaml`](automation.yaml) an die **tatsächlichen Werte** an

### Typische Event-Daten für ZHA
| Aktion | `event_type` | `command` | `action` |
|--------|--------------|-----------|----------|
| Tastendruck | `zha_event` | `"on"` oder `"toggle"` | `"on"` oder `"toggle"` |
| Drehring links | `zha_event` | `"rotate"` | `"rotate_left"` |
| Drehring rechts | `zha_event` | `"rotate"` | `"rotate_right"` |

### Für Zigbee2MQTT
Ersetze `zha_event` durch `zigbee2mqtt_event` und passe die `event_data` an:
```yaml
event_data:
  device_id: DEIN_SCHALTER_DEVICE_ID
  endpoint: 1
  cluster_id: 6  # On/Off Cluster
  command: 1     # Toggle
```
Für Drehring:
```yaml
cluster_id: 8   # Level Control Cluster
command: 0      # Move with On/Off
args: [0, 50]   # Richtung (0=links, 1=rechts), Schrittweite
```

---

## 💡 Tipps für weitere Tasten

Um die **Tasten 2, 3 und 4** für andere Lampen zu nutzen:
1. **Kopiere die Automation** und benenne sie um (z. B. `"Zigbee Handschalter - Taste 2 - Wohnzimmer-Lampe"`)
2. Ändere im kopierten Script:
   - `command: "button_1"` → `command: "button_2"` (oder ähnlich, je nach Schalter)
   - `entity_id: light.dach_lampe_tv` → die jeweilige Lampe (z. B. `light.wohnzimmer_lampe`)

---

## 📝 Hinweise
- Die Dimm-Schrittweite und Übergangszeit kannst du später über die **Input Helfer** anpassen, ohne die Automation zu ändern
- Falls du **Zigbee2MQTT** statt **ZHA** nutzt, passe die `event_type` und `event_data` entsprechend an

---

## 🔗 Nützliche Links
- [Home Assistant Automation Dokumentation](https://www.home-assistant.io/docs/automation/)
- [ZHA Event Dokumentation](https://www.home-assistant.io/integrations/zha/#events)
- [Zigbee2MQTT Dokumentation](https://www.zigbee2mqtt.io/)

---

## 🤝 Beitrag leisten
Falls du Verbesserungen oder Erweiterungen hast, erstelle gerne einen **Pull Request** oder öffne ein **Issue** im Repository.

---

**Viel Spaß mit deiner smarten Beleuchtungssteuerung! 💡**