# Deployment guide

This guide covers the current experimental v10 integration. Back up your ESPHome and Home Assistant configuration before testing.

## Prerequisites

- Home Assistant with access to its configuration directory
- ESPHome Dashboard/add-on or ESPHome CLI
- MQTT broker reachable from Home Assistant and the CoreInk (for example, Mosquitto)
- M5Stack CoreInk and a recovery path to previously working firmware
- Local Wi-Fi credentials and ESPHome API encryption key

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
5. Search Developer Tools → States for `todo_3310` entities. Exact entity IDs can vary with HA version and entity registry history.

The package defines five task labels and completion helpers, three wake times and messages, MQTT telemetry, config publishing, task-state synchronization, and a daily reset at wake slot 0. Review that reset behavior before relying on tasks that should not reset each morning.

## Add the dashboard

Create a manual/YAML dashboard and paste `homeassistant/dashboards/todo_3310.yaml`. Check all referenced entities; adjust IDs if your instance generated different ones.

## Configure ESPHome

1. Copy `esphome/coreink_todo_3310.yaml` into your ESPHome configuration directory.
2. Copy the key names from `examples/secrets.example.yaml` into your local `secrets.yaml`, then replace every example value. Never commit the real secrets file.
3. Ensure the MQTT broker hostname/IP is reachable from the CoreInk network.
4. Run ESPHome Validate / `esphome config <file.yaml>` and fix all errors before flashing.
5. After boot, check the MQTT topics and confirm `todo_3310/state/config_applied` is published.

## MQTT topics

- `todo_3310/config`: retained JSON settings (labels, hours, minutes, messages)
- `todo_3310/command/task/<0..4>/done`: retained task-state commands from HA
- `todo_3310/state/task/<0..4>/done`: task-state reports from device
- `todo_3310/state/availability`: online/offline
- `todo_3310/state/last_seen`: last contact
- `todo_3310/state/battery_voltage`, `todo_3310/state/battery_percent`: battery telemetry
- `todo_3310/state/seconds_to_wakeup`: snapshot calculated before sleep

## Safety and troubleshooting

- Do not use a public MQTT broker; use authentication and limit access to your local network.
- If MQTT config changes while the CoreInk sleeps, the device will apply the retained payload only on its next wake/connect.
- If HA entities are missing, validate the package, restart HA and inspect logs and the entity registry.
- If firmware validation fails, do not flash; check ESPHome version compatibility, MQTT callbacks and JSON lambda code.
- The integration is experimental and has not yet been confirmed by a real ESPHome build and end-to-end hardware test.
