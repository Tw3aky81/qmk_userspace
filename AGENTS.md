# SplitKB QMK Userspace Context

This directory is a specialized QMK Userspace for **SplitKB Halcyon** keyboards. It allows for modular keyboard firmware builds (Kyria, Elora, Ferris, Lily58, Corne) using shared "Halcyon modules" for features like encoders, displays, and trackpads.

## Project Structure

- **`keyboards/splitkb/halcyon/`**: Contains keyboard-specific keymaps and rules.
  - `elora/`, `kyria/`, `ferris/`, `lily58/`, `corne/`
- **`users/halcyon_modules/`**: The core modular logic.
  - `splitkb/halcyon.c`: Main implementation for module synchronization and initialization.
  - `splitkb/config.h`: Key matrix and layout macro definitions (`LAYOUT_elora_hlc`, etc.).
  - `hlc_encoder/`, `hlc_tft_display/`, `hlc_cirque_trackpad/`: Specific module implementations.
- **`layouts/`**: Shared layout JSON files.
- **`qmk.json`**: Build target definitions for `qmk userspace-compile`.

## Building and Running

This project requires a `vial-qmk` environment.

- **Setup**:
  1. `qmk setup -H <path_to_vial_qmk>`
  2. `qmk config user.overlay_dir="$(realpath .)"`
- **Compile Single Target**:
  ```bash
  qmk compile -kb splitkb/halcyon/elora/rev2 -km tw3aky81_vial -e HLC_TFT_DISPLAY=1 -e TARGET=my_firmware
  ```
  or via the wrapper Makefile:
  ```bash
  make splitkb/halcyon/elora/rev2:tw3aky81_vial -e HLC_TFT_DISPLAY=1 -e TARGET=my_firmware
  ```
- **Compile All Targets**:
  ```bash
  qmk userspace-compile
  ```

## Development Conventions

- **Modular Logic**: Shared code resides in `users/halcyon_modules/splitkb/`. Avoid duplicating logic in individual keymaps.
- **Custom Layout Macros**: Use the `_hlc` suffixed macros (e.g., `LAYOUT_elora_hlc`) which reserve extra matrix positions for module inputs (like encoders).
- **RGB Matrix**: Layer-specific RGB logic should be implemented in `keymap.c` using `layer_state_set_user` and `rgb_matrix_indicators_user`.
- **Split Transaction**: Uses a custom `MODULE_SYNC` transaction to synchronize state between the master and slave halves.

## Elora Rev2 LED Index Map

Physical layout viewed from above, with stagger simplified. Numbers are global
RGB Matrix LED indices, not keycodes or matrix coordinates. The source of truth
is Elora's `g_led_config` in `users/halcyon_modules/splitkb/halcyon.c`, interpreted
with `LAYOUT_elora_hlc` in `users/halcyon_modules/splitkb/config.h`.

```text
LEFT HALF                               RIGHT HALF

36 35 34 33 32 31                       68 69 70 71 72 73
30 29 28 27 26 25                       62 63 64 65 66 67
24 23 22 21 20 19                       56 57 58 59 60 61
18 17 16 15 14 13 12 11           48 49 50 51 52 53 54 55
         10  9  8  7  6           43 44 45 46 47
```

Orientation using QWERTY key positions:

| Physical positions | LED indices in the same order |
| --- | --- |
| Q W E R T | 29 28 27 26 25 |
| A S D F G | 23 22 21 20 19 |
| Z X C V B | 17 16 15 14 13 |
| Y U I O P | 62 63 64 65 66 |

Underglow LEDs are **0–5** on the left and **37–42** on the right.
Physical LED indices remain unchanged when key assignments change in Vial.

For the `tw3aky81_vial` Function layer's F-key arrangement:

| Keys | Physical positions | LED indices in the same order |
| --- | --- | --- |
| F1 F2 F3 F4 | Z X C V | 17 16 15 14 |
| F5 F6 F7 F8 | A S D F | 23 22 21 20 |
| F9 F10 F11 F12 | Q W E R | 29 28 27 26 |

**Split indicator handling:** `rgb_matrix_indicators_user()` runs on both halves.
In the current vial-qmk checkout, `rgb_matrix_set_color()` does not reject
left-half indices on the right half, which can cause mirrored highlights.
Guard left-only highlight calls with `if (is_keyboard_left())` and right-only
calls with `if (!is_keyboard_left())`, retaining the global indices above.
Use physical handedness, not USB master status, for these checks.

## Key Files for Reference

- `users/halcyon_modules/splitkb/halcyon.c`: Master/Slave synchronization logic.
- `users/halcyon_modules/splitkb/config.h`: Matrix row/column and layout definitions.
- `keyboards/splitkb/halcyon/elora/keymaps/tw3aky81_vial/keymap.c`: Advanced example with layer-specific RGB indicators.
