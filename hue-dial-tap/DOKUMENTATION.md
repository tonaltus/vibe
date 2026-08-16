# Dokumentation: Zigbee2MQTT-Schalter in Home Assistant

**Ziel:** Ein funktionierendes Toggle-Script für einen Zigbee2MQTT-Schalter (z. B. Hue Tap Dial), das:
1. **Speicherbar** ist (korrekte YAML-Syntax)
2. **Funktioniert** (richtige Trigger-Werte aus den Logs)
3. **Beiden Geräten zugeordnet** wird (Schalter + Lampe)

---

## 📌 Wichtigste Erkenntnisse

### 1. **Trigger-Typ: `platform: device` (nicht `mqtt`!)**
- **Warum?**
  Die `device`-Plattform verknüpft die Automation **automatisch mit dem Gerät** (z. B. `Dach-tap-dial`).
  MQTT-Triggers (`platform: mqtt`) ordnen die Automation **nur der Ziel-Entität** (z. B. Lampe) zu.

- **Syntax:**
  ```yaml
  trigger:
    - platform: device
      device_id: afbc0fec6dfd2c36ba252295054f7188  # Geräte-ID des Schalters
      domain: mqtt
      type: action
      subtype: button_1_press  # Exakter Wert aus den Logs!
  ```

---

### 2. **Action-Werte aus den Logs verwenden**
- **Problem:**
  Jeder Schalter sendet **unterschiedliche Action-Werte** (z. B. `button_1_press`, `dial_rotate_left_step`).
  Diese müssen **exakt** im `subtype`-Feld des Triggers stehen.

- **Lösung:**
  Prüfe die **Zigbee2MQTT-Logs** (z. B. `button_1_press` statt `button_1_press_release`).
  Beispiel aus den Logs:
  ```
  [16.8.2026, 14:24:15] z2m:mqtt: MQTT publish: topic 'zigbee2mqtt/Dach-tap-dial/action', payload 'button_1_press'
  ```
  → **`subtype: button_1_press`** (nicht `_release`!).

---

### 3. **Korrekte Action-Syntax**
- **Falsch:**
  ```yaml
  action:
    - action: light.toggle  # ❌ "action:" gehört nicht hierhin!
      target:
        entity_id: light.dach_lampe_tv
  ```

- **Richtig:**
  ```yaml
  action:
    - service: light.toggle  # ✅ "service:" ist korrekt!
      target:
        entity_id: light.dach_lampe_tv
  ```

---

## 📜 Minimales funktionierendes Script (Toggle)

```yaml
alias: "Toggle Dach-Lampe-TV mit Dach-tap-dial"
description: >
  Toggle für Dach-Lampe-TV mit Geräte-Trigger.
  Wird automatisch dem Schalter und der Lampe zugeordnet.

trigger:
  - platform: device
    device_id: afbc0fec6dfd2c36ba252295054f7188  # Geräte-ID des Schalters
    domain: mqtt
    type: action
    subtype: button_1_press  # Exakter Wert aus den Logs!

action:
  - service: light.toggle
    target:
      entity_id: light.dach_lampe_tv

mode: single
```

---

## 🔍 Fehlersuche

| **Problem** | **Ursache** | **Lösung** |
|-------------|-------------|------------|
| Script wird nicht gespeichert | YAML-Syntaxfehler (z. B. `action:` statt `service:`) | Korrekte Syntax verwenden |
| Toggle funktioniert nicht | Falscher `subtype`-Wert | `subtype` aus den Logs prüfen |
| Keine Gerätezuordnung | `platform: mqtt` statt `platform: device` | `platform: device` + `device_id` nutzen |

---

## 💡 Zusammenfassung der 3 Kriterien

| **Kriterium** | **Umsetzung** | **Beispiel** |
|--------------|---------------|--------------|
| **Speicherbar** | Korrekte YAML-Syntax | `service: light.toggle` |
| **Funktioniert** | Richtige `subtype`-Werte | `subtype: button_1_press` |
| **Gerätezuordnung** | `platform: device` + `device_id` | `device_id: afbc0fec6dfd2c36ba252295054f7188` |

---

**Hinweis:** Diese Dokumentation beschränkt sich auf die **relevanten Punkte für Toggle-Scripts**. Für erweiterte Funktionen (z. B. Dimm) sind zusätzliche Trigger und Aktionen nötig.