# FS25 "True AI Tracks" — throttle fix (scan ran every frame)

A tested one-line fix for the [True AI Tracks](https://www.farming-simulator.com/mod.php?mod_id=318473)
mod (v2.2.0.0, by KCHARRO) for Farming Simulator 25, offered to the author for adoption —
published here as a **patch you apply to your own downloaded copy**. This repository distributes
**no mod files**.

## The problem

`scripts/aiGround.lua` throttles its vehicle scan with:

```lua
lastCheckTime = lastCheckTime + (dt or 0.016)
if lastCheckTime < (SETTINGS.CHECK_INTERVAL / 1000) then return end
```

In FS25, `update(dt)` receives `dt` in **milliseconds** — a ~20 ms frame passes `dt ≈ 20`. The
accumulator counts milliseconds but is compared against `0.15` (seconds), so the intended
150 ms throttle trips on the very first frame, every frame. The full `g_currentMission.vehicles`
scan — with recursive implement and wheel processing — runs ~50× more often than designed,
a constant per-frame cost that grows with vehicle count (worst with several AI helpers active).

## The fix

Compare milliseconds to milliseconds:

```lua
lastCheckTime = lastCheckTime + (dt or 16)
if lastCheckTime < SETTINGS.CHECK_INTERVAL then return end
```

## Measured results

Two 10-minute CapFrameX captures, same savegame and scenario (Riverbend Springs, 2–3 AI helpers
working, CPU-bound machine):

| | bugged | fixed |
| --- | --- | --- |
| median frametime | 19.27 ms | 18.67 ms |
| average fps | 50.8 | 52.6 |
| p99 frametime | 27.0 ms | 25.8 ms |

≈0.6 ms per frame recovered with a handful of vehicles; scales with vehicle count. Track visuals
are unchanged — the scan still runs at the author's intended 150 ms cadence.

The companion mods (True AI Tracks: NEXAT / CRAWLERS) contain no periodic code and need no
changes.

## Applying the patch

Requires the mod already installed from the official ModHub (v2.2.0.0).

```powershell
git clone https://github.com/talontownsend/fs25-true-ai-tracks-fix
cd fs25-true-ai-tracks-fix
.\tools\Apply-Patch.ps1
```

The script backs up your zip (`*.pre-patch.bak`), verifies the hunks match the expected mod
version (any mismatch aborts without changes), edits the script inside your copy, and rebuilds
the zip in place. Decline ModHub update prompts afterwards, or the patch is overwritten — which
is also how you adopt an official fix later: just accept the update.

Patched zips hash differently than ModHub's: fine in singleplayer; multiplayer requires all
players and the server to use the same zip.

## Rights & takedown

The mod remains the property of its author, KCHARRO. The patch contains the minimal code
fragments needed to describe and apply the fix — standard bug-report practice — and nothing
here enables using the mod without downloading it from the official ModHub yourself. **If you
are the author and want anything changed or removed, open an issue and it will be done
promptly.** The goal of this repository is to get the fix into your official release, nothing
more.

The apply script and documentation are MIT-licensed (see LICENSE); the patch files include
fragments of the author's code, which remain under their rights.
