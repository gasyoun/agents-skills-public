## After provisioning (what the user sees on the device)

Once `m5-onboard go` finishes at the `DONE` banner, the device is ready to use on its own:

- **Power.** Slide the switch on the right edge of the Cardputer-Adv to turn it on. Same switch turns it off. The board runs off its internal LiPo when unplugged; USB-C charges it.
- **Boot.** A short boot log scrolls, then the launcher menu appears automatically. The menu lists every `.py` in `/flash/apps/` plus the top-level `/flash/*.py` entries.
- **Navigation.** Arrow keys (or the keyboard's trackpoint-style cursor keys) scroll the menu; Enter launches the highlighted app; ESC returns to the launcher from inside an app.
- **Event WiFi auto-connect.** The bundle's `main.py` connects to a hard-coded event WiFi (SSID `cardputer`) on every boot and shows the result on the LCD before the launcher menu appears. Credentials live in `buddy/device/wifi_event.py`; the connect is best-effort and the launcher always continues even if the connect fails. If you're using this bundle outside the event, edit `wifi_event.py` or remove the `_connect_wifi_with_splash()` call from `main.py`.
- **Claude Buddy over BLE.** First time only: in Claude Desktop, **Help → Troubleshooting → Enable Developer Tools** (one-time, persists across launches). Then **Developer menu → Hardware Buddy → Connect**. BLE works regardless of the WiFi state — the link to Claude.app is local.
- **Getting back to UIFlow.** The buddy bundle ships only a `main.py` at `/flash/` (no replacement `boot.py`), so the stock UIFlow `boot.py` is never touched and there's no `boot_uiflow.py` backup to restore. Revert by removing our `main.py` from the device REPL: `os.remove('/flash/main.py')` followed by `machine.reset()`. UIFlow's stock launcher takes over on the next boot. To start completely fresh including the firmware, re-run the skill without `--apps`.

