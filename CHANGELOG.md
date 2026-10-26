# Changelog

## v0.1.0 — initial repository packaging

- Package the v10 ESPHome/MQTT integration.
- Add Home Assistant package and Lovelace dashboard.
- Add deployment guide, architecture notes, example secret names and Git ignore rules.
- Mark compiler/hardware validation as pending.


## Unreleased — display language

- Fixed device texts in German and English, chosen with the new `input_select.todo_3310_language` in HA and sent as `language` in the config. Defaults to German.
- `clock_3310` gets a "Display language" select (German default) and German labels and weekday names.
- Fixed the clock showing the wrong weekday (ESPHome counts from 1 = Sunday).
- German default titles, finish messages and example tasks in the firmware; German defaults set once by a new HA automation on first start.
- HA helpers no longer use `initial:`, so titles, messages and start times survive HA restarts.
- Uppercase daypart titles now also uppercase ä, ö and ü.
- Mockup: switch between German and English display texts.
- Dashboard view icon `mdi:checklist` (not in Material Design Icons) replaced by `mdi:format-list-checks`.
- Deployment guide: optional NFC tag on the case that opens the dashboard view through the Companion app.
- GitHub Pages start page opens the demo instead of a rendered copy of the README; the demo page gets a short intro, a setup-guide link and a "how it works" strip. README shows two demo screenshots and a prominent demo link.
- Battery icon fills to the battery level; the percentage on top switches between black and white at the fill edge so it stays readable at any level. The mockup has a battery slider to try it.

## dayparts and HA to-do lists
- Each daypart (morning, midday, evening) now has its own list of up to five tasks. The device shows the list of the active daypart, wakes at each daypart start and resets that list once per day.
- Tasks are managed in three Home Assistant Local To-do lists; new example dashboard with a to-do card per daypart, device status tiles and schedule settings.
- New MQTT contract: config version 2 with per-daypart slots, `todo_3310/command/slot/<n>` (consumed by the device) and `todo_3310/state/slot/<n>`. The per-task topics and HA helpers are removed; see the upgrade notes in `docs/DEPLOYMENT.md`.
- A finished list is locked: UP/DOWN no longer dismiss the finish card, PRESS cannot uncheck tasks, and TOP undo only covers the last toggle of the current wake.
- Smaller daypart title so it no longer touches the separator line; footer bar and text moved up so the text clears the bottom edge of the panel.
- Firmware reads the PCF8563 on boot and wakes 20 s after each daypart start; plays a short jingle when a list is finished.
- Before deep sleep the device draws a sleep image with a forced full refresh: no selection bar, footer `NEXT hh:mm` / `SIDE: WAKE`. This clears ghosting from partial refreshes; the boot draw waits for the first battery reading instead of redrawing for it.
- Status bar: wake time larger and centered, battery percentage inside the battery icon.
- Finish card shows the message of the finished daypart (defaults "Have a great day!", "Enjoy your afternoon!", "See you tomorrow!"); progress segments are hidden once the list is done.
- Fixed TOP undo not reporting the restored state to Home Assistant.
- `clock_3310` gets API/OTA, an explicit timezone and reads the RTC on boot.
- Both firmware files pass `esphome config` and `esphome compile` with ESPHome 2026.10.
- The README links the GitHub Pages demo directly; the separate mockup README and the outdated v10 notes are folded into it. Demo links point to GitHub, so they also work on Pages.
- API secret renamed to `todo_3310_api_key`; example secrets cover both variants.

## todo_3310 / clock_3310
- Added generic `esphome/todo_3310.yaml` firmware and renamed the Home Assistant package/dashboard to `todo_3310`.
- Added always-on `esphome/clock_3310.yaml` firmware without a deep-sleep component.
- Reworked the browser mockup to switch between the checklist and a live browser-local clock, with CoreInk-style 200×200 layout.
- Added `docs/BACKLOG.md` with per-task timing, wake-cause-aware e-paper refresh, validation and hardware-test items.
- Mockup screen is drawn as a 200×200 SVG from the firmware coordinates so it scales with the device frame.
- Strike-through on completed tasks is measured with `get_text_bounds` instead of fixed per-slot widths, so it fits renamed tasks.
- When every task is done, PRESS no longer unchecks tasks; it brings the finish card back. TOP undo, the morning reset and Home Assistant can still reopen tasks.
- Translated device UI strings, example tasks/messages, Home Assistant helper names, dashboard and docs to English. The MQTT sensors get new English names, so new installs get entity IDs such as `sensor.todo_3310_battery_mqtt`; existing installs keep their old entity IDs and need the dashboard entity IDs adjusted.
- Firmware compile and hardware validation are still pending.
