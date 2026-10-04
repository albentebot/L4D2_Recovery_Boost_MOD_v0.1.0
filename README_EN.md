# Survivor Recovery Boost 0.1.0 — Mod

Auto-regeneration every second, plus faster healing, defibrillation and revival. Defaults apply to both human and Bot survivors simultaneously, using the game's native item, rescue-check and consumption rules.

| Feature | Default |
| --- | --- |
| Auto-regen | +1 permanent HP per second, cap 100 |
| First aid kit | 2-second use bar — applies to self-heal and healing teammates |
| Defibrillator | 1-second use bar, +0.5 second revive delay after the bar completes |
| Reviving downed teammate | 1-second bar |

This is a **test build**: it has passed syntax checking, logic simulation and VPK-content verification, but has **not yet been play-tested inside L4D2**. Animations, interrupt decisions, adrenaline and other mods may affect actual durations; the values in the table are the configured base timings.

## Installation

Exit the game, then place `survivor_recovery_boost.vpk` into:

```text
Steam/steamapps/common/Left 4 Dead 2/left4dead2/addons/
```

Enable **Survivor Recovery Boost (Regen and Fast Actions)** under the "Add-ons" menu. Launch a single-player Campaign/Realism run, or host a local server yourself.

In chat press **Y** and type `!rb` to check status; in the console use `say !rb` — do **not** type `!rb` directly. The status display shows the *actual* read-back duration of healing, defibrillator and revive bars.

Install **only one** VPK of this mod — the source folder does **not** need to go into the game. No SourceMod, Metamod or other scripting libraries are required. It works standalone, or together with the earlier *Bot TeamPlus 0.1.2* and *Survivor Hazard Shield 0.1.0*; just keep those two VPKs as well.

## Regeneration Rules

- Regenerates **permanent** HP (not temp HP). The cap is the smaller of the configured cap and the player's native max HP, so it will never reduce bonus HP granted by other mods.
- When temporary HP + permanent HP would exceed the cap, only the *temporary HP that overlaps with the newly added permanent HP* is trimmed. Example: 50 perm + 50 temp becomes 51 perm + 49 temp; the remaining temp HP still decays normally per game rules.
- Regen **pauses** while dead, dying, incapacitated or hanging from a ledge. It will **not** auto-stand, revive or clear black-and-white status. A survivor grabbed by a Special Infected but still standing continues to regen.
- Every second all survivors are checked — humans and Bots both benefit. Infected **do not** regen. If the server stutters, regen does **not** "catch up" by granting all missed ticks at once.
- First aid kits and defibrillators still require normal operation and are still consumed — no free items are granted. The speed-up is a server-side base duration; actions already in progress when the value changes will finish with the old timing.

## Chat Commands

Available to **all** players:

```text
!rb
```

The following commands are restricted to the **local host**:

```text
!rboff                         Disable regen and speed-ups; the disabled state is saved
!rbon                          Enable again
!rbreload                      Re-read configuration from disk
!rbset regen_amount 2          +2 permanent HP per second
!rbset regen_amount 0          Disable only regen; keep action speed-ups
!rbset medkit_seconds 1.5      Healing takes 1.5 seconds
!rbset defib_seconds 1         Defibrillator bar 1 second
!rbset defib_return_seconds 0.5 Revive delay after defib bar completes
!rbset revive_seconds 1        Reviving 1 second
!rbset regen_cap 100           Regen cap 100
```

When entering these commands from the console, always prepend `say `. Settings are saved and reloaded immediately, and persist across map changes.

## Configuration File

On first launch the file `left4dead2/ems/recovery_boost/settings.txt` is generated, one `key=value` pair per line. After manual editing, type `!rbreload` or restart the map.

| Key | Default | Allowed Range |
| --- | --- | --- |
| enabled | 1 | 0 or 1 |
| regen_amount | 1 | 0–100 integer, HP per second |
| regen_cap | 100 | 1–100 integer |
| medkit_seconds | 2.0 | 0.2–30 seconds |
| defib_seconds | 1.0 | 0.2–30 seconds |
| defib_return_seconds | 0.5 | 0–10 seconds |
| revive_seconds | 1.0 | 0.2–30 seconds |

Missing or invalid entries fall back to defaults. On a **dedicated server** the VPK must be installed server-side; this version authorizes chat management commands only to the local host, so dedicated servers should edit the config file directly and restart the map. Joining official or someone else's server with only a local install has **no effect**.

## Compatibility & Disabling

Only activates in standard Campaign (`coop`) and Realism (`realism`) modes. On enable the original durations are recorded; on disable, round-end or map-change an attempt is made to restore them. If a map or another mod changed the same Convar later, **this package will not overwrite that newer change** on shutdown.

When used together with the **default Bot TeamPlus** config, the game's native 1-second revive will usually finish *before* Bot TeamPlus's 2-second quick-revive hook. If you want Bots to strictly use this package's revive timing, run `!btpset revive_seconds 0` in chat — that makes the Bot enhancement fall back to the game's vanilla completion check. This command is **not required** to use the default speed-up.

- `!rboff` disables **only** this mod;
- `!btpoff` disables **only** the Bot enhancement;
- **neither** disables the standalone acid-fire damage-reduction package.

To fully uninstall this package, exit the game, remove its VPK and restart. HP already gained through regen is **not** deducted.

Other **regen mods** will **stack** their effects. Other **action-speed mods** or custom campaigns may overwrite the same Convars. Use `!rb` to verify the *actual* variable values. For a first test it is recommended to enable only these three mods together.

## Verification & Game Checklists

On the development side, the following has been completed:

- Squirrel 3.2 syntax check on both scripts; game version and engine extensions differ, so this cannot replace actually loading the mod in-game.
- Simulated per-second regen, no catch-up after stutter, human/Bot parity, HP cap, temp-HP overlap trimming, and exclusion of downed/hanging/dead/infected targets.
- Simulated application of all four speed Convars, single timer on repeated start, restore on disable, preservation of external changes, missing-variable warning, config validation and host permission gates.
- Using published game-event dispatcher code to verify chat parsing, blank handling, deduplication of repeated registrations, recovery after event-table rebuild; simulated three-mod chat and independent toggle coexistence.
- Standalone VPK reader performing CRC and byte-for-byte source validation on every packed file.

**In-game play-testing has not yet been done.** Suggested test sequence:

1. Take damage in a local campaign and observe per-second regen.
2. Test medkit, defib and revive separately — confirm items are consumed normally and interrupted actions do not "instantly finish".
3. Test temp-HP from pills/adrenaline, downed-state pause, map change, and `!rboff` restoring original speeds.
4. Finally add third-party campaigns and other mods.

If problems occur, keep the console lines containing `[RecoveryBoost ERROR]`, the map name and a list of other mods loaded. Repeated script exceptions will auto-disable this mod and attempt to restore durations.

## Source

Source lives in `source/`. Run `python build_vpk.py` to re-pack — Python 3 is required, with no third-party packaging dependencies. Source is licensed **GPL-3.0-or-later**, the license is included.

API references: [Valve VScript API](https://developer.valvesoftware.com/wiki/Left_4_Dead_2/Scripting/Script_Functions), [Game Convar List](https://github.com/Stabbath/L4D2-Decompiled/blob/master/Misc%20Stuff/commoncvars.txt). No other authors' game mods or external runtime libraries are bundled.