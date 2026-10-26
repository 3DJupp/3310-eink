# Changelog

## v0.1.0 — initial repository packaging

- Package the v10 ESPHome/MQTT integration.
- Add Home Assistant package and Lovelace dashboard.
- Add deployment guide, architecture notes, example secret names and Git ignore rules.
- Mark compiler/hardware validation as pending.


## Unreleased — todo_3310 / clock_3310
- Added generic `esphome/todo_3310.yaml` firmware and renamed the Home Assistant package/dashboard to `todo_3310`.
- Added always-on `esphome/clock_3310.yaml` firmware without a deep-sleep component.
- Reworked the browser mockup to switch between the checklist and a live browser-local clock, with CoreInk-style 200×200 layout.
- Added `docs/BACKLOG.md` with per-task timing, wake-cause-aware e-paper refresh, validation and hardware-test items.
- Mockup screen is drawn as a 200×200 SVG from the firmware coordinates so it scales with the device frame.
- Strike-through on completed tasks is measured with `get_text_bounds` instead of fixed per-slot widths, so it fits renamed tasks.
- When every task is done, PRESS no longer unchecks tasks; it brings the finish card back. TOP undo, the morning reset and Home Assistant can still reopen tasks.
- Translated device UI strings, example tasks/messages, Home Assistant helper names, dashboard and docs to English. The MQTT sensors get new English names, so new installs get entity IDs such as `sensor.todo_3310_battery_mqtt`; existing installs keep their old entity IDs and need the dashboard entity IDs adjusted.
- Firmware compile and hardware validation are still pending.
