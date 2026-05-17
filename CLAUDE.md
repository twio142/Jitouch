# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jitouch is a macOS system utility written in Objective-C that expands multi-touch gesture support for MacBook trackpads and Magic Mouse/Magic Trackpad. It consists of two separately built components.

## Build

There are no automated build scripts. Build order matters: the agent must be built first because the preference pane embeds Jitouch.app.

1. Open `jitouch/Jitouch/Jitouch.xcodeproj` in Xcode and build (Release configuration for best performance). This outputs `Jitouch.app` into the `prefpane/` folder.
2. Open `prefpane/Jitouch.xcodeproj` in Xcode and build. This produces `Jitouch.prefPane`.
3. Double-click `Jitouch.prefPane` to install into System Preferences.

There are no automated tests.

## Architecture

The codebase has two distinct components with a shared settings format.

### Background Agent (`jitouch/Jitouch/`)

Runs as a background process. Key files:

- **`Gesture.m`** (4300+ lines) -- Core of the application. Handles multi-touch input via private macOS C APIs (`MTPoint`, `MTReadout`), recognizes gesture patterns, and dispatches actions in `doCommand()`. All gesture execution logic lives here.
- `JitouchAppDelegate.m` -- App lifecycle, event tap setup, accessibility permission handling.
- `KeyUtility.m` -- Keyboard event synthesis.
- `Settings.m` -- Reads/writes the shared `NSUserDefaults` settings.

### Preference Pane (`prefpane/`)

A `NSPreferencePane` plugin loaded by System Preferences. Key files:

- **`TrackpadTab.m`**, **`MagicMouseTab.m`**, **`RecognitionTab.m`** -- Each ~30,000 lines. These are the three main tabs handling gesture configuration UI. The `loadActionButton` method populates the action dropdown; `change:` handles user selection; `commitCommandSheet:` writes the final command dictionary to settings.
- `GesturePreviewView.m` (40,000+ lines) -- Renders gesture preview illustrations.
- `JitouchPref.m` -- Top-level pane controller.
- `Settings.m` -- Shared settings layer (duplicated from agent, kept in sync).

### Settings / Command Dictionary Format

Gesture actions are stored as `NSDictionary` objects in `NSUserDefaults`. Each gesture maps to a command dict with keys such as:
- `"OpenFile"` -- path to open
- `"OpenURL"` -- URL string
- `"ExecuteShellCommand"` -- shell command string

The `doCommand()` function in `Gesture.m` inspects these keys with an if/else chain to decide what action to execute.

## Scripts

- `scripts/bump-version.sh` -- Updates version strings and creates a git tag.
- `scripts/sign-installer.sh` -- Signs and notarizes the installer package for distribution.
