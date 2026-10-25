# 3310-eink — CoreInk todo & clock for Home Assistant

Retro Nokia 3310-inspired monochrome e-paper projects for the M5Stack CoreInk, ESPHome, Home Assistant and MQTT: `todo_3310` (scheduled checklist with deep sleep) and `clock_3310` (live clock, powered continuously without deep sleep).

> **Status: experimental.** The project has not yet been validated with a real ESPHome build or end-to-end CoreInk hardware test. Run ESPHome Validate / `esphome config` before flashing.

## Integration design

- Five configurable task labels and task completion states.
- Three daily wakeup times, each with a display message.
- MQTT retained configuration at `todo_3310/config`, applied when the device reconnects.
- Task-state commands and reports under `todo_3310/command/task/<0..4>/done` and `todo_3310/state/task/<0..4>/done`.
- Battery voltage/percentage, MQTT availability, last-seen timestamp and seconds-to-wakeup snapshot.
- ESPHome API remains available for diagnostics/OTA while the device is awake and connected.

## Repository layout

- `esphome/todo_3310.yaml` — generic checklist firmware
- `homeassistant/packages/todo_3310.yaml` — HA helpers, MQTT entities and automations
- `homeassistant/dashboards/todo_3310.yaml` — manual Lovelace dashboard
- `examples/secrets.example.yaml` — example secret keys; use real values only in local `secrets.yaml`
- `docs/DEPLOYMENT.md` — installation steps
- `docs/ARCHITECTURE.md` — MQTT contract and sync behavior
- `docs/V10_NOTES.md` — current scope and limitations
- `mockup/index.html` — browser-based UI mockup (open locally)

## Quick start

1. Configure an MQTT broker reachable by Home Assistant and the CoreInk.
2. Copy `homeassistant/packages/todo_3310.yaml` to your HA `config/packages/` folder.
3. Ensure packages are enabled under your existing `homeassistant:` section in `configuration.yaml`:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
   Merge this into an existing `homeassistant:` block; do not create a duplicate key.
4. Validate and restart Home Assistant.
5. Add `homeassistant/dashboards/todo_3310.yaml` to a manual dashboard in YAML mode.
6. Copy `esphome/coreink_todo_3310.yaml` into ESPHome and configure local secrets.
7. Run ESPHome Validate / `esphome config` and resolve errors before flashing.
8. On connection, verify MQTT state and `todo_3310/state/config_applied`.

See [Deployment guide](docs/DEPLOYMENT.md) and [Architecture](docs/ARCHITECTURE.md). For the visual concept, open [the browser mockup](mockup/index.html) or read [mockup notes](mockup/README.md). A GitHub Pages workflow is included; after its first successful run, the public preview should be available at https://3djupp.github.io/3310-eink/.

Funding links are available via Ko-fi and Buy Me a Coffee (see GitHub's **Sponsor** button).

## Known limitations

- Exactly five task slots and three daily schedule slots are supported.
- Independent due times per task are not implemented yet; the current firmware uses shared daily wake slots and keeps all tasks in the list.
- `seconds_to_wakeup` is calculated before sleep and is not a live countdown.
- A sleeping ESP32 cannot receive an immediate update; retained settings apply on the next wake/connect.
- Offline conflicting edits do not have a robust revision/merge protocol.
- Display refresh, sleep/wakeup behavior and battery estimates still require hardware testing. RTC-wake full refresh is not yet guaranteed by the current driver configuration.

## Security

Never commit real `secrets.yaml`, Wi-Fi credentials, MQTT passwords, API encryption keys or certificates. Keep the broker authenticated and restricted to your local network.

## License

MIT. See [LICENSE](LICENSE).


## Firmware variants

- **`esphome/todo_3310.yaml`** — generic, time-aware task list. Example tasks should be replaceable and have their own due times / wake scheduling.
- **`esphome/clock_3310.yaml`** — clock display using local time, intended for continuous USB or power-adapter operation; no deep sleep.
- The bathroom checklist for children is only a possible real-world example of `todo_3310`, not the product name or the fixed task set.

## Display refresh and wake-up backlog

- RTC/timer wake-up must always trigger a full e-paper refresh.
- Pin/button wake-up must not force a full refresh on every interaction; use the existing partial-refresh/progress strategy where safe.
- Respect each task's configured time independently. Do not collapse all tasks into one shared wake-up time.
- The browser preview is a layout approximation. Verify exact pixel placement, font metrics, and refresh behavior against a real CoreInk before release.
