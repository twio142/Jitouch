# Implementation Plan: Custom Shell Command Support

**Branch**: `001-shell-command` | **Date**: 2026-05-17 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/001-shell-command/spec.md`

## Summary

Add a "Run Shell Command..." gesture action that lets users map any trackpad/mouse gesture
to an arbitrary shell command. The command is stored in the gesture's command dictionary
under the key `"ExecuteShellCommand"` and executed via `NSTask` when the gesture fires.
Changes span all three pref pane tab files (UI) and `Gesture.m` (execution). No new
frameworks or abstractions are needed; the implementation follows the existing
`"Open File..."` / `"Open Website..."` patterns precisely.

## Technical Context

**Language/Version**: Objective-C (MRC — manual retain/release; no ARC in the agent)

**Primary Dependencies**: Foundation, AppKit, NSTask — all standard macOS frameworks
already in use; no new dependencies

**Storage**: `NSUserDefaults` via the existing command dictionary (`NSDictionary`) format;
new key `"ExecuteShellCommand"` (NSString) added per gesture

**Testing**: None — manual verification only (Constitution Principle V)

**Target Platform**: macOS (same as existing Jitouch support matrix)

**Project Type**: macOS desktop system utility (background agent + preference pane plugin)

**Performance Goals**: Shell command dispatch MUST complete within the gesture callback
without perceptible latency; `NSTask launchedTaskWithLaunchPath:arguments:` is
fire-and-forget and meets this requirement

**Constraints**: Objective-C only; no ARC; no new frameworks; Three-Tab Rule (all three
tab files must be updated identically); `NSAutoreleasePool` required in agent

**Scale/Scope**: ~7 files modified; net change approximately 50-80 lines total

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
|-----------|--------|-------|
| I. Build Order | ✅ Pass | No change to build system or project files |
| II. Settings Parity | ✅ Pass | `"ExecuteShellCommand"` is a per-gesture dict key, not a `Settings.m` key; no Settings.m changes needed |
| III. Command Dictionary Contract | ✅ Pass | All four integration points implemented: `loadActionButton`, `change:`, `commitCommandSheet:`, `doCommand()` |
| IV. Follow Existing Patterns | ✅ Pass | Using `NSAlert` + `NSTextField` accessory (same as URL entry flow), `NSTask` (same as AppleScript file execution), `NSAutoreleasePool` (same as `OpenURL` block) |
| V. Manual Verification | ✅ Pass | Verification scenario defined in quickstart.md and original PLAN.md |

## Project Structure

### Documentation (this feature)

```text
specs/001-shell-command/
├── plan.md              # This file
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
└── tasks.md             # Phase 2 output (/speckit-tasks — NOT created here)
```

### Source Code (repository root)

```text
prefpane/
├── TrackpadTab.h        # Add NSString *shellCommand; ivar
├── TrackpadTab.m        # loadActionButton, change:, commitCommandSheet:, showCommandSheet
├── MagicMouseTab.h      # Add NSString *shellCommand; ivar
├── MagicMouseTab.m      # Same changes as TrackpadTab.m
├── RecognitionTab.h     # Add NSString *shellCommand; ivar
└── RecognitionTab.m     # Same changes as TrackpadTab.m

jitouch/Jitouch/
└── Gesture.m            # Add ExecuteShellCommand branch to doCommand() if/else chain
```

**Structure Decision**: No new files. All changes are in-place edits to existing files
in the two existing Xcode project trees.
