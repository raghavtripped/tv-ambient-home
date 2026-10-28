# Examples

The debloat list applied to my TV, with the packages disabled on that firmware, grouped and commented.
It started from the lists in [CleanYourTV](https://github.com/Gathin23/cleanyourtv/tree/main/examples), which
has lists for 200+ other models; check there first if you have a different TV.

| Folder | TV | OS | Notes |
|---|---|---|---|
| `xiaomi-mitv-moor2/` | Xiaomi MiTV-MOOR2 (venom, MediaTek) | Google TV 11 | Mi ad row and PatchWall, Mi analytics, MiLink, store demo; setup wizard off so a third-party launcher can own Home |

Apply it with `TV_HOST=<tv ip> tools/disable.sh examples/xiaomi-mitv-moor2/debloat.txt` and undo it with
`TV_HOST=<tv ip> tools/restore.sh examples/xiaomi-mitv-moor2/debloat.txt com.google.android.apps.tv.launcherx`.
