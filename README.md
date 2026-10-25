# 3310-eink — CoreInk checklist for Home Assistant

Retro Nokia 3310-inspired monochrome e-paper checklist built around the M5Stack CoreInk, ESPHome, Home Assistant and MQTT.

> **Status: experimental.** The project has not yet been validated with a real ESPHome build or end-to-end CoreInk hardware test. Run ESPHome Validate / `esphome config` before flashing.

## Integration design

- Five configurable task labels and task completion states.
- Three daily wakeup times, each with a display message.
- MQTT retained configuration at `bad3310/config`, applied when the device reconnects.
- Task-state commands and reports under `bad3310/command/task/<0..4>/done` and `bad3310/state/task/<0..4>/done`.
- Battery voltage/percentage, MQTT availability, last-seen timestamp and seconds-to-wakeup snapshot.
- ESPHome API remains available for diagnostics/OTA while the device is awake and connected.

## Repository layout

- `esphome/coreink_bad3310.yaml` — device firmware
- `homeassistant/packages/bad3310.yaml` — HA helpers, MQTT entities and automations
- `homeassistant/dashboards/bad3310.yaml` — manual Lovelace dashboard
- `examples/secrets.example.yaml` — example secret keys; use real values only in local `secrets.yaml`
- `docs/DEPLOYMENT.md` — installation steps
- `docs/ARCHITECTURE.md` — MQTT contract and sync behavior
- `docs/V10_NOTES.md` — current scope and limitations
- `mockup/index.html` — browser-based UI mockup (open locally)

## Quick start

1. Configure an MQTT broker reachable by Home Assistant and the CoreInk.
2. Copy `homeassistant/packages/bad3310.yaml` to your HA `config/packages/` folder.
3. Ensure packages are enabled under your existing `homeassistant:` section in `configuration.yaml`:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
   Merge this into an existing `homeassistant:` block; do not create a duplicate key.
4. Validate and restart Home Assistant.
5. Add `homeassistant/dashboards/bad3310.yaml` to a manual dashboard in YAML mode.
6. Copy `esphome/coreink_bad3310.yaml` into ESPHome and configure local secrets.
7. Run ESPHome Validate / `esphome config` and resolve errors before flashing.
8. On connection, verify MQTT state and `bad3310/state/config_applied`.

See [Deployment guide](docs/DEPLOYMENT.md) and [Architecture](docs/ARCHITECTURE.md). For the visual concept, open [the browser mockup](mockup/index.html) or read [mockup notes](mockup/README.md).

Funding links are available via Ko-fi and Buy Me a Coffee (see GitHub's **Sponsor** button).

## Known limitations

- Exactly five task slots and three daily schedule slots are supported.
- Tasks are not filtered by time of day; all remain in the shared list.
- `seconds_to_wakeup` is calculated before sleep and is not a live countdown.
- A sleeping ESP32 cannot receive an immediate update; retained settings apply on the next wake/connect.
- Offline conflicting edits do not have a robust revision/merge protocol.
- Display refresh, sleep/wakeup behavior and battery estimates still require hardware testing.

## Security

Never commit real `secrets.yaml`, Wi-Fi credentials, MQTT passwords, API encryption keys or certificates. Keep the broker authenticated and restricted to your local network.

## License

MIT. See [LICENSE](LICENSE).
