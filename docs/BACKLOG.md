# Backlog — todo_3310 / clock_3310

## P0 — Required before first reliable release
- [ ] Validate/compile the new generic `esphome/todo_3310.yaml` firmware.
- [ ] Validate/compile `esphome/clock_3310.yaml`: actual local time, continuous operation, no deep sleep.
- [ ] Implement independent per-task times / due windows and calculate the next wake-up from all active task schedules.
- [ ] Force a full e-paper refresh after RTC/timer wake-up.
- [ ] Keep pin/button wake-up on the partial-refresh path when possible; do not force full refresh on every pin wake.
- [ ] Run ESPHome validation/compile for both variants and document untested hardware behavior honestly.

## P1 — Display fidelity and device behavior
- [ ] Match browser preview to the firmware's 200×200 layout: status bar, title/count, fixed task rows, progress segments and footer.
- [ ] Test partial-refresh progression, selection inversion, completed-task marks and ghosting on real CoreInk hardware.
- [ ] Verify time synchronization, timezone, midnight rollover and missed task windows.
- [ ] Verify sleep scheduling, button wake, task persistence and battery reporting for `todo_3310`.
- [ ] Verify clock updates while powered, Wi-Fi reconnect behavior and display refresh interval for `clock_3310`.

## P2 — Home Assistant and documentation
- [ ] Rename package, dashboard, MQTT topics and entity IDs to generic `todo_3310` naming.
- [ ] Document bathroom tasks / children's routines only as an optional practical example.
- [ ] Enable GitHub Pages with Source = GitHub Actions and verify the deployment.
- [ ] Add setup instructions and example secrets for both variants.
