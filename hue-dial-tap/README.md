# Hue Dial Tap - Home Assistant Automation

Dieser Ordner enthält **Home Assistant Automationen** für den **Zigbee2MQTT Schalter "Dach-tap-dial"** zur Steuerung der Lampe **"Dach-Lampe-TV"**.

---

## 📁 Dateien in diesem Ordner

| Datei | Beschreibung | Status |
|-------|--------------|--------|
| [`automation.yaml`](automation.yaml) | **Vollständiges Script** (Toggle + Dimm) | ✅ Für Zigbee2MQTT angepasst |
| [`test-toggle.yaml`](test-toggle.yaml) | **Minimales Test-Script** (nur Toggle zur Fehlersuche) | ✅ Neu erstellt |

---

## 🔍 Fehlersuche: Schritt-für-Schritt-Anleitung

**Falls das Script nicht funktioniert, folge dieser Anleitung:**

### **🔹 Schritt 1: Teste das minimale Script (nur Toggle)**
1. **Kopiere den Inhalt von [`test-toggle.yaml`](test-toggle.yaml)** in den **YAML-Editor** deines Home Assistant Automationseditors.
2. **Speichere die Automation** und drücke **Taste 1** deines Schalters.

   **Mögliche Ergebnisse:**
   | Ergebnis | Bedeutung | Nächster Schritt |
   |----------|-----------|------------------|
   | ✅ Lampe schaltet um | Events werden korrekt empfangen | Gehe zu [Schritt 2](#-schritt-2-teste-das-vollständige-script) |
   | ❌ Nichts passiert | Problem bei Event-Integration oder Entity-ID | Gehe zu [Schritt 3](#-schritt-3-prüfe-die-event-integration) |

---

### **🔹 Schritt 2: Teste das vollständige Script**
Falls der Toggle funktioniert:
1. **Kopiere den Inhalt von [`automation.yaml`](automation.yaml)** in den YAML-Editor.
2. **Speichere und teste:**
   - Taste 1 → Lampe sollte toggle
   - Drehring links → Lampe sollte dunkler werden
   - Drehring rechts → Lampe sollte heller werden

   **Falls der Drehring nicht funktioniert:**
   - Prüfe, ob die **`action`-Werte** in den Logs mit den Werten im Script übereinstimmen.
   - Falls nicht, passe die `action`-Werte im Script an (siehe [Dokumentation der Actions](#-dokumentation-der-actions)).

---

### **🔹 Schritt 3: Prüfe die Event-Integration**
Falls **auch der Toggle nicht funktioniert**, prüfe folgende Punkte:

#### **1. Ist die Zigbee2MQTT-Integration aktiv? **
- Gehe zu: **Einstellungen → Geräte & Dienste → Integrationen**
- Suche nach **"Zigbee2MQTT"**
  - ✅ **Aktiv?** → Weiter zu Punkt 2
  - ❌ **Nicht aktiv?** → Füge die Integration hinzu (URL: `http://[DEINE_Z2M_IP]:8080`)

#### **2. Wird der Schalter in Home Assistant erkannt? **
- Gehe zu: **Einstellungen → Geräte & Dienste → Entitäten**
- Suche nach deinem Schalter (z. B. `sensor.dach_tap_dial` oder `light.dach_tap_dial`)
  - ✅ **Vorhanden?** → Weiter zu Punkt 3
  - ❌ **Nicht vorhanden?** → Prüfe die Verbindung zwischen Zigbee2MQTT und Home Assistant

#### **3. Stimmt der `friendly_name` im Script? **
- Dein Schalter hat in Zigbee2MQTT den Namen: **`Dach-tap-dial`** (aus deinen Logs)
- Im Script muss stehen:
  ```yaml
  device:
    friendly_name: Dach-tap-dial
  ```
  - ✅ **Stimmt?** → Weiter zu Punkt 4
  - ❌ **Stimmt nicht?** → Passe den `friendly_name` im Script an

#### **4. Stimmt die Entity-ID der Lampe? **
- Gehe zu: **Einstellungen → Geräte & Dienste → Entitäten**
- Suche nach deiner Lampe und notiere die **genaue `entity_id`** (z. B. `light.dach_lampe_tv`)
- Im Script muss stehen:
  ```yaml
  entity_id: light.dach_lampe_tv
  ```
  - ✅ **Stimmt?** → Weiter zu Punkt 5
  - ❌ **Stimmt nicht?** → Passe die `entity_id` im Script an

#### **5. Kommen die Events in Home Assistant an? **
- Gehe zu: **Entwicklerwerkzeuge → Ereignisse**
- **Lösche alle Events** (Button "Löschen")
- Drücke **Taste 1** deines Schalters
- Suche nach Events mit:
  - **`event_type: zigbee2mqtt_event`**
  - **`data.device.friendly_name: Dach-tap-dial`**
  - **`data.action: button_1_press`**

  | Ergebnis | Bedeutung | Lösung |
  |----------|-----------|--------|
  | ✅ Event gefunden | Events kommen an | Prüfe das Script auf Tippfehler |
  | ❌ Kein Event gefunden | Events kommen nicht in HA an | Prüfe die Zigbee2MQTT-Integration |

---

### **🔹 Schritt 4: Prüfe die Zigbee2MQTT-Logs**
Falls keine Events in Home Assistant ankommen:
1. Öffne das **Zigbee2MQTT-Dashboard** (z. B. `http://[DEINE_IP]:8080`)
2. Gehe zu **"Logs"** oder beobachte die **Echtzeit-Logs**
3. Drücke **Taste 1** und drehe den Ring
4. **Kopiere die Log-Einträge** und vergleiche sie mit den erwarteten Werten:
   - Taste 1: `action: button_1_press`
   - Drehring links: `action: dial_rotate_left_step` oder `brightness_step_down`
   - Drehring rechts: `action: dial_rotate_right_step` oder `brightness_step_up`

---

## 📌 Dokumentation der Actions

Dein Schalter sendet folgende **`action`-Werte** (aus deinen Logs):

| **Aktion** | **Mögliche `action`-Werte** |
|------------|-------------------------------|
| Taste 1 drücken | `button_1_press` |
| Taste 1 loslassen | `button_1_press_release` |
| Drehring links (Schritt) | `dial_rotate_left_step` |
| Drehring links (schnell) | `dial_rotate_left_fast` |
| Drehring links (langsam) | `dial_rotate_left_slow` |
| Drehring links (alternativ) | `brightness_step_down` |
| Drehring rechts (Schritt) | `dial_rotate_right_step` |
| Drehring rechts (schnell) | `dial_rotate_right_fast` |
| Drehring rechts (alternativ) | `brightness_step_up` |

---

## 🛠️ Häufige Probleme & Lösungen

| **Problem** | **Ursache** | **Lösung** |
|-------------|-------------|------------|
| Keine Events in HA | Zigbee2MQTT-Integration fehlt | Integration in HA hinzufügen |
| Schalter nicht erkannt | Schalter nicht mit Z2M gepaart | Schalter in Z2M neu paaren |
| Falsche `action`-Werte | Abweichung von Dokumentation | Logs prüfen und Script anpassen |
| Lampe reagiert nicht | Falsche `entity_id` | Entity-ID in HA prüfen |
| Dimm-Funktion funktioniert nicht | Falsche `action`-Werte für Drehring | Alle Varianten im Script abdecken |

---

## 💡 Tipps für weitere Tasten

Um die **Tasten 2, 3 und 4** für andere Lampen zu nutzen:
1. Kopiere das Script und benenne es um (z. B. `"Zigbee2MQTT - Taste 2 - Wohnzimmer-Lampe"`)
2. Ändere im Script:
   - `action: button_1_press` → `action: button_2_press` (usw.)
   - `entity_id: light.dach_lampe_tv` → `entity_id: light.deine_andere_lampe`

---

## 📚 Nützliche Links
- [Home Assistant Automation Dokumentation](https://www.home-assistant.io/docs/automation/)
- [Zigbee2MQTT Dokumentation](https://www.zigbee2mqtt.io/)
- [Zigbee2MQTT Home Assistant Integration](https://www.home-assistant.io/integrations/zigbee2mqtt/)

---

## 🤝 Beitrag leisten
Falls du Verbesserungen oder Erweiterungen hast, erstelle gerne einen **Pull Request** oder öffne ein **Issue** im Repository.

---

**Viel Erfolg mit deiner smarten Beleuchtungssteuerung! 💡**