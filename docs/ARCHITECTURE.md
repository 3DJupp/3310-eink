# Architecture and synchronization

## Source of truth

Home Assistant is intended to be the configuration UI for labels, wake times, messages and task state. ESPHome owns hardware behavior: e-paper, physical buttons, battery sensing, sleep/wakeup, local task interaction and MQTT transport.

## MQTT contract

### Configuration: `bad3310/config`

Retained JSON object with this shape:

```json
{
  "version": 1,
  "labels": ["Vitamin D", "Jod", "Omega 3", "Zähneputzen", "Babyspa"],
  "hours": [6, 12, 18],
  "minutes": [0, 0, 0],
  "messages": ["Guten Morgen", "Mittagszeit", "Bis später"]
}
```

The current firmware supports exactly five labels and three schedule slots. The CoreInk is usually asleep; retained configuration is available when it reconnects, but cannot wake the sleeping hardware by itself.

### Task state

- Device state: `bad3310/state/task/<slot>/done`
- HA command: `bad3310/command/task/<slot>/done`
- Payloads for task completion are expected as `ON`/`OFF` in the initial integration.

### Telemetry

- `bad3310/state/availability`: `online` or `offline`
- `bad3310/state/last_seen`: last contact timestamp
- `bad3310/state/config_applied`: configuration acknowledgement/version marker
- `bad3310/state/battery_voltage`: voltage
- `bad3310/state/battery_percent`: estimated percent
- `bad3310/state/seconds_to_wakeup`: snapshot computed before deep sleep

## Wakeup and task behavior

The schedule has three daily slots. The current reset behavior resets all task statuses at the first schedule slot each day. Daypart-specific task filtering is not implemented. The display order is rebuilt at boot/wakeup so unfinished tasks appear before completed tasks, then remains stable while the device is awake.

## Conflict handling: known gap

The initial version does not implement a robust per-task revision/merge protocol. If the user edits a task locally while the device is disconnected and also changes that task in HA, the last command/state arriving may win. Before relying on simultaneous offline edits, add a revision or event queue protocol and test it under disconnect/reconnect conditions.

## Deep sleep and availability

`online` only means the device currently has an MQTT connection; the device is expected to go offline/sleep for long periods. A Home Assistant availability timeout should therefore not be interpreted as a hardware fault by itself. Last-seen time and the last reported wakeup/telemetry are historical values.
