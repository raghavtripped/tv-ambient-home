# AGENTS.md

Instructions for coding agents (Claude Code, Codex, Gemini CLI, Cursor, Copilot and others) working in this
repository. People should start with [README.md](README.md). This project is built on
[CleanYourTV](https://github.com/Gathin23/cleanyourtv) by Gathin; see [CREDITS.md](CREDITS.md).

## First: which job are you doing?

| You were asked to | Do this |
|---|---|
| Clean up a TV: debloat it, install the launcher, push a theme | Read [PROMPT.md](PROMPT.md) and follow it exactly, on the real TV. Use the tools in `tools/` and the launcher in `launcher/` rather than writing new ones; the only file you need to change is `launcher/app/src/main/res/values/config.xml`, or a theme's `config.json`, for this TV. |
| Change this repo: the launcher, an add-on, a tool, docs | Read on from [Repo map](#repo-map). |

### Working on a TV: the rules that are not negotiable

[PROMPT.md](PROMPT.md) is the procedure. Its safety rules, in brief:

- **Disable, never uninstall.** `pm disable-user --user 0`, one package per shell call. No
  `pm uninstall` of a preload, no root, nothing flashed. Every step must be undone by `tools/restore.sh`.
- **Back up first.** Package list, disabled list, `getprop` and settings into `backup/` before the first
  change. `tools/disable.sh` also writes its own record in `~/.cleanyourtv/`; keep the list you applied.
- **Leave alone what the TV needs to work:** the TV app and input services behind HDMI and the tuner,
  the input/source menu, remote and Bluetooth pairing, the on-screen keyboard, Play Store and Play
  Services. When a package name says nothing, read `dumpsys package <name>`, then ask.
- **Never start a factory or test package** (`mediatek.factorymenu.ui` and the like), not even to look.
  On some boards that flips a factory-mode flag the shell cannot undo.
- **Keep the Home guard.** The stock launcher is disabled only after the new launcher is installed and
  confirmed as Home (`dumpsys activity activities | grep mFocusedApp`). On Google TV, check
  `dumpsys wifi | grep -o 'cname=[^ ]*'` before disabling setupwraith: if it owns the saved Wi-Fi
  network, disabling it takes the TV off Wi-Fi and ADB. `tools/disable.sh` and
  `launcher/build-and-install.sh` already enforce this. Use `FORCE_HOME=1` or `FORCE_SETUPWRAITH=1` only in
  the situation the script's own message describes, and only with the person's agreement.
- **Ask before anything the family uses.** Streaming apps, the voice assistant, Chromecast, Miracast and
  the firmware updater go into one checklist question; disable only what the person does not tick.
- **Someone may be watching.** Don't jump to Home, change the volume or restart an app that is playing
  (`dumpsys media_session`) without asking, and put things back afterwards.
- **Disable in small batches** and screenshot between them (`tools/tv.py --screenshot shots/01.png`).
  Live TV, HDMI and some streaming apps give empty screenshots; that is the TV, not damage.
- **An injected key is not proof** for a remote button that goes through an accessibility helper: ask the
  person to press the real button. `am force-stop com.gathin.tvhome` switches those helpers off.
- **Close ADB afterwards.** USB debugging and Network/ADB debugging off, authorisations revoked, reboot,
  then `tools/adb-closed.py <tv ip>` must report the port closed. Port 5555 is plaintext.
- **Stop and say so** when the TV refuses: `pm disable-user` refused for every package, a sideload that
  fails verification, or an SDK below the launcher's `minSdk` 26. Do not retry in a loop.

The person's backups (`backup/`), screenshots (`shots/`), notes (`work/`) and the ADB key
(`tools/adbkey/`) are listed in `.gitignore` and must never be committed.

## Repo map

| Path | What it is |
|---|---|
| `PROMPT.md` | The step-by-step procedure for an agent cleaning a TV; the safety rules live here. |
| `README.md` | Front page: what's in it, the TV, set-up, undo, lessons learnt. |
| `CREDITS.md` | The original project, its contributors, and what this project adds. |
| `examples/` | `examples/xiaomi-mitv-moor2/debloat.txt`, the list applied to this TV. Indexed in `examples/README.md`. |
| `docs/` | `docs/TROUBLESHOOTING.md`, `docs/REMOTE.md`, `docs/modules.md` and `docs/specs/card.md` (module cards), `docs/app-library.md`, `docs/sky.md`, `docs/watch-next.md`. |
| `launcher/` | The launcher, package `com.gathin.tvhome` (kept so it installs over the existing one): plain Java, AndroidX RecyclerView and DynamicAnimation, `minSdk` 26, no `INTERNET` permission. |
| `themes/` | `default` and `aurora`: `config.json`, `screenshot.jpg`, optional wallpaper. Keys in `themes/README.md`. |
| `addons/` | `ambilight`, `weather`, `phoneremote`: module-card apps, each its own Gradle project. Indexed in `addons/README.md`. |
| `tools/` | The ADB client and scripts: `tv.py`, `b64push.py`, `relay.py` (guide: `tools/relay.md`), `disable.sh`, `restore.sh`, `recovery.py`, `check.py`, `privacy.py`, `theme.sh`, `adb-closed.py`, `bench.py`, `race_calendar.py`, with PowerShell twins. Each script's usage is in its first lines. |
| `tests/` | Python `unittest` suite: the list format, README tables, themes, add-ons, the Home guard and the tools against fake TVs. |
| `.github/workflows/` | `ci.yml` (tests, launcher build, APK size gate) and `addons.yml`. |

## Rules for changes

- **Add-ons stay add-ons.** They talk to the launcher only through the module-card contract
  (`docs/modules.md`), have a Permissions section in their README, and a row in `addons/README.md`.
- **The launcher stays small and offline:** no `INTERNET` permission, no telemetry, no new dependencies,
  no background polling. New behaviour is a `config.xml` key read in `Config.java`.
- **No prebuilt APKs or binaries** in the repo. Build from source.
- **Logic that can be tested without a TV gets a JVM unit test** under `launcher/app/src/test/java/com/gathin/tvhome/`
  (or the add-on's own unit tests).
- **Verify on the real TV**, not only in tests: `tools/tv.py --screenshot shots/after.png`, and for the LED strip a
  photo of the wall, since "strip ready" in the log does not prove it is lit.
- **Keep the credits.** Do not remove the original copyright in `LICENSE` or the attribution in `README.md` and
  `CREDITS.md`.

## Before committing

| Check | Command or rule |
|---|---|
| Tests, as CI runs them | `python3 -m unittest discover -s tests -v` after `python3 -m venv tools/venv` and `tools/venv/bin/pip install -r tools/requirements.txt`. On `main`, CI sets `CLEANYOURTV_STRICT=1`. |
| Launcher, if you touched `launcher/` | JDK 17, Android SDK platform 34, build-tools 34.0.0. In `launcher/`: `./gradlew --no-daemon testReleaseUnitTest lintRelease assembleRelease`. Lint must stay at zero errors. |
| APK size gate | `launcher/app/build/outputs/apk/release/app-release.apk` must be under 512,000 B (500 KB). Linux: `stat -c %s`, macOS: `stat -f %z`. |
| Add-ons, if you touched one | In `addons/<name>/`: `./gradlew --no-daemon testReleaseUnitTest lintRelease assembleRelease`. |
| Identifiers | Scan the diff: no IP addresses, Wi-Fi names, account emails, serial numbers, Android IDs, MAC addresses, device names or ADB keys. Use `<tv ip>` or `192.168.1.50` in docs, `AA:BB:CC:DD:EE:FF` for a MAC. |

## Privacy and screenshots

- The launcher's top bar shows the Wi-Fi network name. Blur Wi-Fi names, people's names and addresses, or crop
  them out, before a screenshot goes into the repo.
- Never commit TV Settings screenshots that show accounts, email addresses or device names.
- `tools/tv.py --probe`, `getprop` and backups contain the serial number and device names. Quote only the model,
  Android version and firmware build.

## Commit style

Titles are `area: what changed`, in plain words: `Launcher: stock home redirect no longer covers an app that just
opened`, `addons: Ambient light waits after power-on`. The area is the folder the change is in (`Launcher`,
`addons`, `tools`, `docs`, `examples`, `themes`, `tests`, `README`). The body is one or two sentences on what
changed and how it was tested on the TV.
