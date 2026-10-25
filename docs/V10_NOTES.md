# BAD 3310 – Home Assistant + ESPHome v10 (MQTT-Konfiguration)

## Enthalten
- ESPHome-Firmware-Basis, MQTT-Config-Receiver, fünf editierbare Labels, drei konfigurierbare tägliche Wakeup-Slots, lokale Bedienung und Akku-Messung.
- Home Assistant package mit Helpers, MQTT-Sensoren und Automationen für Konfiguration und Status-Sync.
- Lovelace-Dashboard YAML.
- Telemetrie enthält Akku, MQTT-Verfügbarkeit, letzten Kontakt und die beim Einschlafen berechneten Sekunden bis zum nächsten Wakeup (kein live herunterzählender Timer).

## Verhalten und Grenzen
- Fünf Aufgaben-Slots, frei benennbar; der Status wird zwischen HA und CoreInk abgeglichen.
- Drei tägliche Wakeup-Slots, jeweils mit eigener Nachricht.
- Zeitfenster sind tägliche Uhrzeiten, keine beliebigen Wochentags-/Kalenderregeln. Task-Tageszeiten sind noch nicht mit einer Filterlogik verbunden; Aufgaben werden weiter als gemeinsame Liste angezeigt.
- Bei Offline-Gerät wird retained Konfiguration beim nächsten Connect ausgeliefert.
- Die Firmware braucht für neue Logik/Funktionen weiterhin ein OTA-Update; Labels und Zeiten erfordern keinen Neucompile.

## Vor dem Flashen
Die Dateien sind experimentell und wurden nicht durch einen realen ESPHome-Build und End-to-End-Hardwaretest bestätigt. Insbesondere MQTT-Callbacks/JSON-Lambda und die installierte ESPHome-Version müssen durch `esphome config` bzw. Dashboard-Validate verifiziert werden. Erst danach flashen.
