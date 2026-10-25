# CoreInk UI preview

Open [`index.html`](index.html) in a browser to switch between the two conceptual firmware views. The page also links directly to the [todo_3310 YAML](../esphome/todo_3310.yaml) and [clock_3310 YAML](../esphome/clock_3310.yaml):

- `todo_3310`: generic five-task checklist with the CoreInk-style status bar, task rows, progress strip and footer.
- `clock_3310`: browser-local live clock as a preview of the always-on, no-deep-sleep variant.

The screen is an SVG with a 200 × 200 viewBox that reuses the pixel coordinates from the firmware display lambdas, so it scales with the device frame at any window width. Fonts differ from the firmware's Segoe UI bitmap fonts, so text widths are approximate.

The device shell is a CSS illustration inspired by the M5Stack CoreInk, not a product photograph. The page is a layout approximation, not a hardware screenshot or proof that the firmware has compiled or been tested on-device.
