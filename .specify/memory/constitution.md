<!--
SYNC IMPACT REPORT
==================
Version change: (none) → 1.0.0 (initial ratification)

Modified principles: N/A (initial)

Added sections:
  - Core Principles (I–V)
  - Platform Constraints
  - Development Workflow
  - Governance

Removed sections: N/A (initial)

Templates reviewed:
  - .specify/templates/plan-template.md ✅ — "Constitution Check" gate aligns with Principles I–V; no edit needed
  - .specify/templates/spec-template.md ✅ — scope/requirements format compatible; no edit needed
  - .specify/templates/tasks-template.md ✅ — tests marked OPTIONAL; consistent with Principle V; no edit needed
  - .specify/templates/commands/ ✅ — directory not present; no action needed

Deferred TODOs: none
-->

# Jitouch Constitution

## Core Principles

### I. Build Order Is Non-Negotiable

The background agent (`jitouch/Jitouch/Jitouch.xcodeproj`) MUST be built before the
preference pane (`prefpane/Jitouch.xcodeproj`). The prefpane embeds Jitouch.app at build
time; reversing the order produces a broken or stale install. Release builds MUST use the
Release configuration for the agent target.

### II. Settings Parity

`Settings.m` / `Settings.h` are intentionally duplicated between `jitouch/Jitouch/` and
`prefpane/`. Any change to a settings key, default value, or accessor MUST be applied to
both copies in the same commit. A mismatch silently corrupts gesture configuration at
runtime and is hard to diagnose.

### III. Command Dictionary Contract

Gesture actions are encoded as `NSDictionary` objects stored in `NSUserDefaults`. Every
new action type MUST implement all four integration points:

- **UI prompt** in `change:` (TrackpadTab, MagicMouseTab, RecognitionTab)
- **Persistence** in `commitCommandSheet:` (all three Tab files)
- **Display** in `loadActionButton` (all three Tab files)
- **Execution** in `doCommand()` in `Gesture.m`

The dictionary key (e.g., `"ExecuteShellCommand"`) is the shared contract between the
preference pane and the agent. Adding a key on one side without the other MUST NOT happen.

### IV. Follow Existing Patterns

This is a mature, stable codebase. New features MUST use established patterns rather than
introducing new abstractions:

- **Execution**: extend the if/else chain in `doCommand()` in `Gesture.m`
- **UI prompt**: follow the `NSAlert` + `NSTextField` accessory view pattern used by
  `"Open File..."` and `"Open Website..."`
- **Memory**: use `NSAutoreleasePool` / `[pool release]`; ARC is not in use in the agent

No new frameworks, bridging headers, or architectural layers are permitted unless replacing
something demonstrably broken.

### V. Manual Verification Is the Only Gate

No automated test infrastructure exists. All verification is performed manually:

1. Build the agent (Release), then the prefpane.
2. Install the prefPane by double-clicking it.
3. Assign the new action to a gesture in System Preferences → Jitouch.
4. Trigger the gesture and observe the result.

Every feature spec or plan MUST include a concrete manual verification scenario (see
PLAN.md for the reference format: build → configure → trigger → observe artifact).

## Platform Constraints

- **Private APIs**: Multi-touch input uses private macOS C APIs (`MTPoint`, `MTReadout`).
  These are undocumented; changes require empirical testing on target hardware. No public
  alternatives exist.
- **Entitlements**: Jitouch.app requires Accessibility permissions granted explicitly by
  the user. Any new system interaction (e.g., shell execution) MUST be feasible within the
  app's existing entitlements; adding entitlements requires re-signing.
- **Language**: The codebase MUST remain Objective-C. No Swift, no mixed-language targets.

## Development Workflow

- **Branch strategy**: Feature work happens on named branches (e.g., `feat/shell`).
  `PLAN.md` at the repo root documents the active in-progress feature.
- **Three-Tab Rule**: Any UI change to gesture action types MUST be applied identically
  across TrackpadTab, MagicMouseTab, and RecognitionTab. Divergence produces inconsistent
  behavior across device types.
- **Versioning**: `scripts/bump-version.sh` MUST be used to update version strings and
  create git tags. No manual version edits.
- **Distribution**: `scripts/sign-installer.sh` handles signing and notarization and MUST
  run before any public release artifact is published.

## Governance

This constitution supersedes all other development practices for Jitouch. Amendments
require: (1) a clear rationale, (2) a version bump per the policy below, and (3) a
consistency check across plan-template, spec-template, and tasks-template.

**Versioning policy**:
- MAJOR: Removal or redefinition of an existing principle
- MINOR: New principle or section added
- PATCH: Clarification or wording refinement

All implementation plans MUST include a Constitution Check gate verifying compliance with
Principles I–V before implementation begins. Use `CLAUDE.md` for runtime build and
architecture guidance.

**Version**: 1.0.0 | **Ratified**: 2026-05-17 | **Last Amended**: 2026-05-17
