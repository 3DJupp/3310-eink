# BAD 3310 – Home Assistant + ESPHome v10 (MQTT configuration)

## Included
- ESPHome firmware base, MQTT config receiver, five editable labels, three configurable daily wakeup slots, local controls and battery measurement.
- Home Assistant package with helpers, MQTT sensors and automations for configuration and state sync.
- Lovelace dashboard YAML.
- Telemetry includes battery, MQTT availability, last contact and the seconds to the next wakeup calculated when going to sleep (not a live countdown).

## Behavior and limits
- Five freely named task slots; state is synchronized between HA and the CoreInk.
- Three daily wakeup slots, each with its own message.
- Time windows are daily times, not arbitrary weekday or calendar rules. Per-task times of day are not yet tied to any filtering; tasks are still shown as one shared list.
- If the device is offline, the retained configuration is delivered on the next connect.
- New firmware logic or features still need an OTA update; labels and times do not require a recompile.

## Before flashing
These files are experimental and have not been confirmed by a real ESPHome build and end-to-end hardware test. In particular the MQTT callbacks/JSON lambda and the installed ESPHome version must be checked with `esphome config` or Dashboard Validate. Flash only after that.
