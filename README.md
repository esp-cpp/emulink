# emulink

The USB link of the esp-cpp emulators,
[esp-box-emu](https://github.com/esp-cpp/esp-box-emu) and
[tab5-emu](https://github.com/esp-cpp/tab5-emu): one copy of everything both
of them use, as a submodule of each (at `components/emulink`).

- **Firmware** (an ESP-IDF component, `emulink`):
  - `DevLink` ([include/devlink.hpp](include/devlink.hpp)): the espp
    dispatcher on the USB vendor interface, with the espp system, monitor and
    coredump services, and the last panic kept across the reset.
  - `RemoteControl` ([include/remote_control.hpp](include/remote_control.hpp)):
    the test control (module `0x20`, `emu.control`): status, the game list,
    launch / quit, buttons, touch, save states, screenshots, the SD card's
    files, its USB drive mode, the launcher's settings, the updater. Header
    only: the app's `main` includes it, with its own `cart.hpp`, `gui.hpp`
    and `settings.hpp`.
- **[tools/emulink](tools/emulink)**: the command line and Python API on the
  host (`python -m emulink --help` from that directory).
- **[web](web)**: the Emulator Console, the WebUSB page the firmware
  advertises (each emulator's GitHub Pages publish it).

## Using it in an emulator

```cmake
# CMakeLists.txt, before project(): the board component, which provides
# box-emu.hpp (class BoxEmu); box-emu when not set
idf_build_set_property(EMULINK_BOARD_COMPONENT tab5-emu)
```

and `emulink` in the project's component list. The board's `BoxEmu` gives the
link its USB vendor interface (`vendor_write`, `set_vendor_rx`,
`usb_prepare_for_rom_download`), the LVGL lock (`lvgl_mutex`), and the
scripted input (`set_remote_buttons`, `set_remote_touch`, `frame_to_screen`).

## emulink

```sh
cd tools/emulink
python -m emulink devices               # the emulators on USB
python -m emulink status
python -m emulink launch "Dark Forces"
python -m emulink press a               # buttons
python -m emulink tap 640 360           # touch (screen pixels)
python -m emulink drag --frame 100 90 200 90 --ms 400   # in screenshot pixels
python -m emulink screenshot -o shot.png
python -m emulink ota build/app.bin     # the firmware, through the updater
```

`--serial tab5-emu` / `--serial esp-box-emu` picks one when both are plugged in.

## License

MIT
