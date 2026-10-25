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
- Firmware compile and hardware validation are still pending.
