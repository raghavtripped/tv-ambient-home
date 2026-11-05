# Project status

**On hold since 5 November 2026.** Everything below is working on the TV, committed and pushed. Nothing is
half-done in the code. ADB debugging is switched off on the TV with the remote as the last step (step 5 of
[Resuming](#resuming)), so picking this up again starts by switching it back on.

TV: Xiaomi MiTV-MOOR2, Google TV / Android 11, firmware RTM5.220609.003.2674.

## What is done and verified on the TV

| Part | State | How it was checked |
|---|---|---|
| Debloat | 22 packages disabled (not uninstalled), [`examples/xiaomi-mitv-moor2/debloat.txt`](../examples/xiaomi-mitv-moor2/debloat.txt) | `tools/check.py`: nothing drifted |
| Home screen | The launcher owns Home; Google TV home stays enabled for voice-search results and is sent back when it appears on its own | Screenshots; Home button and boot |
| Remote buttons | Home, Settings (quick panel), Input, YouTube, Xiaomi TV button (Live TV) | Injected keys; real-remote check still open |
| Volume acceleration | 1 press = 1 step; quick presses or holding go 2, 4, 8 steps | Unit tests, injected keys |
| Bluetooth pill | Paired devices with connection state in the top bar | Screenshots |
| Ambient light | Strip follows the dominant colour of the picture (also Netflix, Prime and other protected video), beat flashes from the TV's sound, off in standby, back after standby, reboot, full power-off and app update | Photos of the wall, Mac Bluetooth experiments, the owner's confirmation |
| Weather, Phone remote | Tiles open; forecast loads only while open | Screenshots |
| Stock-home redirect fix | No longer covers an app that is still starting (the Hotstar splash hang after the screensaver) | Unit test; not yet seen live |
| Repository | 210 repo tests, 185 launcher tests, 45 Ambient light tests, lint clean, CI green | `python3 -m unittest discover -s tests -v`, Gradle |

## Known limits

- **Standby means offline.** In standby the TV answers no ADB, ping or Wake-on-LAN; only the remote wakes it.
- **The light has no way to see a picture without the TV's colour taking.** Screen capture stays off on this TV
  because it breaks Prime Video's HDCP check.
- **Clearing the Ambient light's data** removes its saved strip address and permissions; the recovery steps are
  in [`addons/ambilight/README.md`](../addons/ambilight/README.md).
- **Memory is tight** (1.8 GB). Apps sometimes start from scratch after the screensaver, which takes a few seconds.
- The launcher APK is about 477 KB, above the original project's 460 KB limit; this repo's CI allows 500 KB.

## Open, all optional

- Watch the stock-home redirect fix live: leave the screensaver with Back and open an app straight away.
- Press the real remote: hold and tap volume, Home, Settings, Input, YouTube; speak a search with the Assistant button.
- Photograph the wall once after the 1 November data reset of the Ambient light (the log says it is connected).
- Approve the Phone remote card in Launcher settings → Module cards, and pair a real phone.
- Decide on DNS: AdGuard blocks ads but answers slowly from here; Cloudflare is about ten times faster with no blocking.

## Ideas not started

- A smart bulb (Tuya local key) that follows the same colour as the strip.
- A second LED strip.
- Pointer speed and icon for the Bluetooth mouse.

## Resuming

1. On the TV: Settings → System → Developer options → **USB debugging** and **Network debugging** on.
2. On the computer, from the repo root: `python3 -m venv tools/venv && tools/venv/bin/pip install -r tools/requirements.txt`
   (once), then `TV_HOST=<tv ip> tools/venv/bin/python -I tools/tv.py --probe`, and accept "Allow USB debugging?" on the TV.
3. Compare the TV with the list: `TV_HOST=<tv ip> tools/venv/bin/python -I tools/check.py examples/xiaomi-mitv-moor2/debloat.txt`
   (add `--fix` after a firmware update).
4. Build what you change: [`AGENTS.md`](../AGENTS.md) has the commands and the rules (tests, size limit, identifier scan).
5. When done, close ADB again: revoke the authorisations, switch both debugging options off, reboot, and check with
   `tools/venv/bin/python -I tools/adb-closed.py <tv ip>`.
