# Deployment guide

This guide covers the current experimental integration. Back up your ESPHome and Home Assistant configuration before testing.

## Prerequisites

- Home Assistant 2024.10 or newer with access to its configuration directory
- ESPHome Dashboard/add-on or ESPHome CLI (both firmware files compile with ESPHome 2026.10)
- MQTT broker reachable from Home Assistant and the CoreInk (for example, Mosquitto)
- M5Stack CoreInk and a recovery path to previously working firmware
- Local Wi-Fi credentials and one ESPHome API encryption key per device

## Create the to-do lists

In Home Assistant open Settings → Devices & services → Add integration → **Local To-do** and create three lists with exactly these names:

| List name           | Entity ID                 | Shown on the CoreInk |
|---------------------|---------------------------|----------------------|
| `todo_3310 Morning` | `todo.todo_3310_morning`  | from the morning start |
| `todo_3310 Midday`  | `todo.todo_3310_midday`   | from the midday start |
| `todo_3310 Evening` | `todo.todo_3310_evening`  | from the evening start |

If HA generated different entity IDs, rename them in the entity settings. Add up to five tasks per list; further items stay in HA but are not shown on the device. You can rename the lists' display names later, as long as the entity IDs stay the same.

## Install the Home Assistant package

1. Configure and start your MQTT broker, and configure Home Assistant's MQTT integration.
2. Copy `homeassistant/packages/todo_3310.yaml` to `<HA config>/packages/todo_3310.yaml`.
3. Ensure packages are included under the existing `homeassistant:` section in `configuration.yaml`:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
   If `homeassistant:` already exists, merge the option into it rather than adding a duplicate key.
4. Validate the Home Assistant configuration and restart.
5. Search Developer Tools → States for `todo_3310` entities.

The package defines daypart start times, titles and finish messages, MQTT telemetry, an "active list" sensor, the scripts that send config and list state to the device, and automations for syncing and the daily reset of each list at its daypart start.

### Upgrading from the five-task version

The per-task helpers `input_text.todo_3310_task_<n>_label` and `input_boolean.todo_3310_task_<n>_done` are gone. Copy your tasks into the new to-do lists, then remove the orphaned helpers in Settings → Devices & services → Entities. The old retained topics `todo_3310/command/task/<n>/done` and `todo_3310/state/task/<n>/done` are no longer used and can be cleared on the broker. The ESPHome secret `coreink_bathroom_todo__encryption_key` is now called `todo_3310_api_key`.

## Add the dashboard

The file `homeassistant/dashboards/todo_3310.yaml` is a single dashboard **view**, not a whole dashboard. Open a dashboard (or create one under Settings → Dashboards), choose the pencil, add a view or open an existing one, then the view's three-dot menu → **Edit in YAML**, and replace everything with the file. Pasting it into the view editor as a whole dashboard (with `views:`) shows an empty view, and HA appends `cards: []`. It uses only built-in cards: one to-do card per daypart with its start time, device status tiles and the schedule and text helpers.

## Configure ESPHome

1. Copy `esphome/todo_3310.yaml` into your ESPHome configuration directory.
2. Put the font file `segoeuithibd.ttf` into a `fonts/` folder next to it, or change the three `font:` entries to a font you have (for example `gfonts://Roboto`). Layout spacing is tuned for Segoe UI metrics.
3. Copy the key names from `examples/secrets.example.yaml` into your local `secrets.yaml`, then replace every example value. Never commit the real secrets file.
4. Check the `timezone` substitution (default `Europe/Berlin`).
5. Ensure the MQTT broker hostname/IP is reachable from the CoreInk network.
6. Run ESPHome Validate / `esphome config todo_3310.yaml` and fix all errors before flashing.
7. After boot, check the MQTT topics and confirm `todo_3310/state/config_applied` is published.

`esphome/clock_3310.yaml` needs only Wi-Fi and `clock_3310_api_key`; its font comes from Google Fonts.

## MQTT topics

- `todo_3310/config`: retained JSON settings (per daypart: title, start time, message, task labels)
- `todo_3310/command/slot/<0..2>`: retained list state from HA, consumed by the device
- `todo_3310/state/slot/<0..2>`: list state reports from the device
- `todo_3310/state/availability`: online/offline
- `todo_3310/state/last_seen`, `todo_3310/state/active_slot`: last contact and shown daypart
- `todo_3310/state/battery_voltage`, `todo_3310/state/battery_percent`: battery telemetry
- `todo_3310/state/seconds_to_wakeup`: snapshot calculated before sleep

See [Architecture](ARCHITECTURE.md) for payloads and sync rules.

## Safety and troubleshooting

- Do not use a public MQTT broker; use authentication and limit access to your local network.
- If the lists change while the CoreInk sleeps, the device applies them on its next wake/connect. Press the side button to wake it right away.
- If the device shows the default example tasks, it has not received a config yet: check `todo_3310/config` on the broker and the HA automation trace of "todo_3310 - Send config to CoreInk".
- If HA entities are missing, validate the package, restart HA and inspect logs and the entity registry.
- If firmware validation fails, do not flash; check the ESPHome version and the compile log.
- The integration is experimental and has not yet been confirmed by an end-to-end hardware test.
