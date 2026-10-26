# Backlog — todo_3310 / clock_3310

## Done
- [x] Validate/compile `esphome/todo_3310.yaml` (ESPHome 2026.10, `esphome config` and `esphome compile`).
- [x] Validate/compile `esphome/clock_3310.yaml`: local time, continuous operation, no deep sleep.
- [x] Different tasks per time of day: one list per daypart, next wake calculated from all daypart starts.
- [x] Full e-paper refresh after RTC/timer wake (the driver's first update after every boot is a full refresh).
- [x] Read the PCF8563 on boot instead of trusting the ESP32 clock after deep sleep.
- [x] Lock a finished list: UP/DOWN/PRESS cannot reopen tasks; TOP undo is limited to the current wake.
- [x] Title and footer text no longer touch the separator line / bottom panel edge.
- [x] Browser preview matches the firmware's 200×200 layout and button behavior.
- [x] Generic `todo_3310` naming for package, dashboard, MQTT topics and entity IDs.
- [x] Bathroom tasks / children's routines documented only as an example.
- [x] GitHub Pages deployment, linked from the README.
- [x] Setup instructions and example secrets for both variants; API/OTA for `clock_3310`.
- [x] Finish jingle when the last open task is checked.
- [x] Manage tasks in Home Assistant to-do lists with an example dashboard.

## P0 — Required before first reliable release
- [ ] Flash both variants on a real CoreInk and confirm boot, display and buttons.
- [ ] Confirm the footer and title positions on the physical panel after this change.

## P1 — Display fidelity and device behavior
- [ ] Test partial-refresh progression, selection inversion, completed-task marks and ghosting on real hardware.
- [ ] Verify time synchronization, timezone, midnight rollover, daypart resets and missed dayparts.
- [ ] Verify sleep scheduling, button wake, task persistence and battery reporting for `todo_3310`.
- [ ] Verify clock updates while powered, Wi-Fi reconnect behavior and display refresh interval for `clock_3310`.
- [ ] Clip or shorten task labels that are wider than a row.
- [ ] Pin/button wake still starts with a full refresh because the panel loses its image in deep sleep; check whether keeping the panel powered is worth the battery cost.

## P2 — Home Assistant and sync
- [ ] Optional weekday rules (for example a different list on weekends).
- [ ] Revision numbers for offline edits on both sides instead of "last change wins".
- [ ] More than five tasks per daypart (scrolling list).
