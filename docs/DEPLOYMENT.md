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

The package defines daypart start times, titles and finish messages, the display language (Deutsch/English), MQTT telemetry, an "active list" sensor, the scripts that send config and list state to the device, and automations for syncing and the daily reset of each list at its daypart start.

On the first start a one-time automation fills in the defaults: 06:00 / 12:00 / 18:00, the titles "Morgens", "Mittags", "Abends" and the messages "Schönen Tag!", "Schönen Nachmittag!", "Bis morgen!". Change them in the dashboard afterwards; the helpers have no `initial:` value, so HA keeps your values across restarts.

### Upgrading from the previous package

The helpers no longer set `initial:` values, which made HA reset titles, messages and start times on every restart. After updating, HA keeps whatever the helpers hold now; the defaults automation does not overwrite them. The new `input_select.todo_3310_language` starts on Deutsch.

### Upgrading from the five-task version

The per-task helpers `input_text.todo_3310_task_<n>_label` and `input_boolean.todo_3310_task_<n>_done` are gone. Copy your tasks into the new to-do lists, then remove the orphaned helpers in Settings → Devices & services → Entities. The old retained topics `todo_3310/command/task/<n>/done` and `todo_3310/state/task/<n>/done` are no longer used and can be cleared on the broker. The ESPHome secret `coreink_bathroom_todo__encryption_key` is now called `todo_3310_api_key`.

## Add the dashboard

The file `homeassistant/dashboards/todo_3310.yaml` is a single dashboard **view**, not a whole dashboard. Open a dashboard (or create one under Settings → Dashboards), choose the pencil, add a view or open an existing one, then the view's three-dot menu → **Edit in YAML**, and replace everything with the file. Pasting it into the view editor as a whole dashboard (with `views:`) shows an empty view, and HA appends `cards: []`. It uses only built-in cards: one to-do card per daypart with its start time, device status tiles and the schedule and text helpers.

### Optional: NFC tag on the case opens the dashboard

The CoreInk cannot read NFC itself, but a passive NFC sticker on or in the case can open the dashboard on a phone. Use the Home Assistant Companion app's deep link instead of a browser URL: it uses the server address configured in the app, so it also works away from home, and it opens the app rather than a browser that would need to log in.

The link is `homeassistant://navigate/<dashboard-path>/<view-path>`. With the view from this repository on the default dashboard that is `homeassistant://navigate/lovelace/todo_3310`; for another dashboard, take the path from its URL in the browser.

- **Android:** write the link as a URI record to the tag with an NFC writer app (for example NFC Tools). Holding the phone to the tag opens the view.
- **iPhone:** iOS does not open custom links from tags in the background. Create a personal automation in the Shortcuts app with the trigger **NFC** (scan the tag), the action **Open URLs** with the link above, and set it to run immediately.

A tag written this way opens the view but does not fire a Home Assistant `tag_scanned` event. To use the same tag for an HA automation as well, write it from HA (Settings → Tags) instead; it then fires the event but does not open this view.

## Configure ESPHome

1. Copy `esphome/todo_3310.yaml` into your ESPHome configuration directory.
2. Put the font file `segoeuithibd.ttf` into a `fonts/` folder next to it, or change the three `font:` entries to a font you have (for example `gfonts://Roboto`). Layout spacing is tuned for Segoe UI metrics.
3. Copy the key names from `examples/secrets.example.yaml` into your local `secrets.yaml`, then replace every example value. Never commit the real secrets file.
4. Pick a free static IP for each device outside your router's DHCP range and set `todo_3310_ip`, `clock_3310_ip`, `wifi_gateway`, `wifi_subnet` and `wifi_dns`. Without DHCP each wake connects faster, which saves battery. To use DHCP instead, delete the `manual_ip:` block under `wifi:`.
5. Check the `timezone` substitution (default `Europe/Berlin`).
6. Ensure the MQTT broker hostname/IP is reachable from the CoreInk network.
7. Run ESPHome Validate / `esphome config todo_3310.yaml` and fix all errors before flashing.
8. After boot, check the MQTT topics and confirm `todo_3310/state/config_applied` is published.

`esphome/clock_3310.yaml` needs only the Wi-Fi secrets (including `clock_3310_ip`) and `clock_3310_api_key`; its font comes from Google Fonts.

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
