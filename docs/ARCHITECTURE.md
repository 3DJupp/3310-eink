# Architecture and synchronization

## Source of truth

Home Assistant is the configuration UI: three to-do lists (one per daypart) hold the tasks, and helpers hold the daypart start times, titles and finish messages. ESPHome owns hardware behavior: e-paper, physical buttons, battery sensing, sleep/wakeup, local task interaction and MQTT transport.

## Dayparts

There are three dayparts (slot 0..2, default morning 06:00, midday 12:00, evening 18:00). The active daypart is the one that started most recently; before the first start of the day, yesterday's latest daypart is still active.

On every boot the firmware reads the PCF8563 RTC (the ESP32's own clock drifts during deep sleep), picks the active daypart and resets its list if that daypart started today and was not reset yet today. Only the active list is shown. Before going to sleep it schedules the next wake 20 seconds after the next daypart start, so a slightly slow RTC still lands in the new daypart.

Home Assistant resets the matching to-do list at the same start time, so both sides agree even when the device is asleep.

## MQTT contract

### Configuration: `todo_3310/config`

Retained JSON published by HA on start, on any change of the lists or helpers, and every ten minutes (renaming or reordering a to-do item does not change the entity state, so the periodic refresh picks that up):

```json
{
  "version": 2,
  "slots": [
    {"title": "Morning", "hour": 6, "minute": 0, "message": "Good morning",
     "tasks": ["Brush teeth", "Take medication", "Get dressed", "Pack school bag"]},
    {"title": "Midday", "hour": 12, "minute": 0, "message": "Lunchtime",
     "tasks": ["Water plants", "Get the mail", "Hang up laundry"]},
    {"title": "Evening", "hour": 18, "minute": 0, "message": "See you later",
     "tasks": ["Tidy the kitchen", "Brush teeth", "Read"]}
  ]
}
```

Only the first five items of each list are sent, each cut to 32 characters. The device stores labels, times and texts in flash so it works without a connection. When labels change it carries done states over by label name, so renaming or reordering a list does not move a check mark to another task. A config identical to the stored one does not redraw the e-paper.

### List state from HA: `todo_3310/command/slot/<slot>`

Retained JSON `{"labels": [...], "done": [true, false, ...]}`, published only when a person changes a list in HA (dashboard, app, voice). The device applies it on its next connect by label name, reports the resulting state and then clears the retained message, so an old command cannot re-check tasks after a later daypart reset. HA's daypart reset also clears it.

### List state from the device: `todo_3310/state/slot/<slot>`

JSON in the same shape, published on connect for all three dayparts, after every toggle or undo on the device, and after applying a command. HA updates the to-do items by label name. These updates are made by an automation, not a person, so they are not echoed back as commands.

### Telemetry

- `todo_3310/state/availability`: `online` or `offline`
- `todo_3310/state/last_seen`: last contact timestamp
- `todo_3310/state/active_slot`: daypart shown at the last connect
- `todo_3310/state/config_applied`: acknowledgement after a config message
- `todo_3310/state/battery_voltage`: voltage
- `todo_3310/state/battery_percent`: estimated percent
- `todo_3310/state/seconds_to_wakeup`: snapshot computed before deep sleep

## Device interaction

- UP/DOWN move the selection, PRESS toggles the selected task, TOP undoes the most recent toggle of the current wake.
- The row order is rebuilt at boot (open tasks first, then completed ones) and stays fixed while the device is awake.
- Once every task of the active list is done, the list is locked: the finish card stays and UP, DOWN and PRESS only extend the awake window. TOP still undoes the last toggle if it happened during this wake; after the next wake that undo is gone. The daypart reset or a change in HA reopens tasks.
- Checking the last open task plays a short jingle.

## E-paper refresh

Deep sleep powers down the panel's image memory, and the ESPHome driver starts every boot with a full refresh. So each RTC wake and each button wake begins with a full refresh, and only the redraws during the awake window (selection, toggles) use partial refreshes, with a full refresh every sixth update (`full_update_every`). A partial refresh straight after a wake is not possible with this panel and driver.

## Conflict handling: known gap

There is no per-task revision protocol. If a task is changed on the device and in HA while they cannot talk to each other, the last change to arrive wins. The device reports its state before it receives a pending command, so for a moment HA may show the device's older state; the report that follows the command brings both back in line.

## Deep sleep and availability

`online` only means the device currently has an MQTT connection; it is expected to be offline/asleep for long periods. A Home Assistant availability timeout is therefore not a hardware fault by itself. Last-seen time and the last reported telemetry are historical values.

## Variants

- `todo_3310` is generic: task labels, completion state and daypart times are configurable. A bathroom checklist for children is only an example use case.
- `clock_3310` is a separate always-on firmware variant. It never enters deep sleep and displays the current local time continuously.
