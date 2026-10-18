---
name: ios-test-hygiene
description: Keep iOS simulator UI-test runs and crash-fuzz runs from dying on a laptop - diagnosing "stopped because the system is running low on memory", simulator process pileups, and the endless macOS crash-report dialog that appears while a fuzz gate deliberately kills your app. Use when xcodebuild UI tests die on 16GB Macs, when you run crash fuzzing, or when a "dialog that never goes away" shows up.
---

# ios-test-hygiene

## Memory wall on a 16 GB Mac

Symptom: a full UI-test sweep dies with `stopped because the system is running low on memory`, repeatedly. (The numbers below come from the author's 16 GB machine: rules of thumb, not constants.)

**Don't be fooled by RSS sums.** Hundreds of simulator daemon processes "adding up to 16 GB" is an over-count: they share memory. The real indicators are `top -l 1 | grep PhysMem` (**unused**) and
`sysctl vm.swapusage`. When unused memory drops to a few hundred MB, the next process to start tends to die.

Levers, in order of effect:
1. **Free memory held by GUI apps** (browser, chat clients, remote desktop). On the author's machine they alone used most of the 16 GB and freeing a couple of GB let the sweep pass.
   **Never close the user's apps from an unattended agent** (it can cut off their remote session or lose unsaved work): ask first, or tell the user which apps to close.
2. **Chunk the tests** (4 -> 2 -> 1 per xcodebuild call). 1 has the lowest peak but adds a boot of roughly a minute per chunk.
3. **Reap between chunks:** `xcrun simctl shutdown all`, then, only if orphans remain, `pkill -9 -f CoreSimulator/Volumes` and `killall -9 launchd_sim`, then boot again. `shutdown` alone can leave orphaned runtime processes.
   **Warning:** the `pkill`/`killall` commands kill every simulator process on the machine (including ones you or Xcode are using) and the pattern can match unrelated processes. Run them only on a machine dedicated to testing, and check what matches first (`pgrep -fl CoreSimulator/Volumes`).

If it still dies: measure free memory first; if there is none, ask the human to close apps. Reducing chunk size will just move the death to the build step.

## The crash dialog that won't dismiss

A fuzz gate that intentionally crashes the app makes macOS queue a "quit unexpectedly" modal per crash. Clicking "ignore" shows the next one immediately.
This changes a **user-wide** setting, so do it on a test machine, or restore it afterwards (separate blocks, so you do not run the restore by accident):

```bash
# 1. turn the modal off (persists until you restore it)
defaults write com.apple.CrashReporter DialogType none
# 2. clear the modals that are already queued
killall UserNotificationCenter
# 3. when the fuzzing is finished, restore the previous behaviour
defaults delete com.apple.CrashReporter DialogType
```
- The owner of the modal is `UserNotificationCenter` (not ReportCrash, which only writes logs).
- In the author's tests `DialogType none` kept writing `.ips` logs to `~/Library/Logs/DiagnosticReports/`. **Keep them:** a crash site (file:line chain) can be pinned from one `.ips` file. Never disable logging to silence the dialog.
- Queue state: `~/Library/Application Support/CrashReporter/*.plist`. Valid `DialogType` values are documented as `basic`, `developer`, `none` and `server`; check `man ReportCrash` on your macOS version.
- Treat step 1 as a precondition of the fuzz gate on a dedicated machine, and step 3 as the cleanup.
