---

description: "Task list template for feature implementation"
---

# Tasks: Custom Shell Command Support

**Input**: Design documents from `specs/001-shell-command/`

**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, quickstart.md ✅

**Tests**: Not requested — manual verification only (Constitution Principle V).

**Organization**: Tasks grouped by user story for independent implementation and testing.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel with another [P] task (different files, no dependency)
- **[Story]**: US1 = Assign Shell Command; US2 = Edit/Replace; US3 = All Device Types

---

## Phase 1: Foundational (Header Changes)

**Purpose**: Add `shellCommand` ivar to all three tab headers before any implementation
files are modified. All three are independent files and can be edited in parallel.

**⚠️ CRITICAL**: Implementation phases depend on these being complete first.

- [x] T001 [P] Add `NSString *shellCommand;` instance variable to interface in `prefpane/TrackpadTab.h`
- [x] T002 [P] Add `NSString *shellCommand;` instance variable to interface in `prefpane/MagicMouseTab.h`
- [x] T003 [P] Add `NSString *shellCommand;` instance variable to interface in `prefpane/RecognitionTab.h`

**Checkpoint**: All three headers updated — implementation phases can now begin.

---

## Phase 2: User Story 1 + 2 — Assign and Edit Shell Command on Trackpad (Priority: P1/P2) 🎯 MVP

**Goal**: A user can assign any shell command to a trackpad gesture, confirm it runs when
triggered, and subsequently edit it.

**Independent Test**: Assign `touch /tmp/jitouch_test_file` to any trackpad gesture,
trigger it, and verify the file is created. Then reassign to a different command and verify
only the new command runs.

### Implementation for US1 / US2

- [x] T004 [P] [US1] Add `"Run Shell Command..."` menu item and `if (shellCommand)` display block (`Run "<cmd>"`) to `loadActionButton` in `prefpane/TrackpadTab.m`
- [x] T005 [P] [US1] Add `else if ([commandDict objectForKey:@"ExecuteShellCommand"])` block with `NSAutoreleasePool` + `NSTask` launch to `doCommand()` in `jitouch/Jitouch/Gesture.m`
- [x] T006 [US1] Add `"Run Shell Command..."` selection handler (NSAlert + NSTextField accessoryView, presented as sheet on commandSheet) to `change:` in `prefpane/TrackpadTab.m`
- [x] T007 [US1] Add `if (shellCommand) [newCommand setObject:shellCommand forKey:@"ExecuteShellCommand"];` and clear `openFilePath`/`openURL` when shellCommand is set, in `commitCommandSheet:` in `prefpane/TrackpadTab.m`
- [x] T008 [US2] Initialize `shellCommand = nil` on new gesture open and load `[oldItem objectForKey:@"ExecuteShellCommand"]` when editing, in `showCommandSheet` in `prefpane/TrackpadTab.m`

**Checkpoint**: US1 and US2 fully functional on Trackpad tab. Build and verify via
`specs/001-shell-command/quickstart.md` steps 1–7 before proceeding.

---

## Phase 3: User Story 3 — Consistent Behavior Across All Device Types (Priority: P3)

**Goal**: Identical shell command functionality on the Magic Mouse and Recognition tabs.

**Independent Test**: Assign shell commands on both MagicMouseTab and RecognitionTab and
verify execution via the same `touch /tmp/...` test on each tab.

### Implementation for US3 — MagicMouseTab and RecognitionTab

MagicMouseTab and RecognitionTab changes mirror Phase 2 exactly. Each column of tasks
(MagicMouseTab vs RecognitionTab) can run in parallel.

- [x] T009 [P] [US3] Mirror T004 (`loadActionButton` changes) in `prefpane/MagicMouseTab.m`
- [x] T010 [P] [US3] Mirror T004 (`loadActionButton` changes) in `prefpane/RecognitionTab.m`
- [x] T011 [P] [US3] Mirror T006 (`change:` handler) in `prefpane/MagicMouseTab.m`
- [x] T012 [P] [US3] Mirror T006 (`change:` handler) in `prefpane/RecognitionTab.m`
- [x] T013 [P] [US3] Mirror T007 (`commitCommandSheet:` persistence) in `prefpane/MagicMouseTab.m`
- [x] T014 [P] [US3] Mirror T007 (`commitCommandSheet:` persistence) in `prefpane/RecognitionTab.m`
- [x] T015 [P] [US3] Mirror T008 (`showCommandSheet` load/init) in `prefpane/MagicMouseTab.m`
- [x] T016 [P] [US3] Mirror T008 (`showCommandSheet` load/init) in `prefpane/RecognitionTab.m`

**Checkpoint**: All three device tabs functional — all user stories independently testable.

---

## Phase 4: Polish & Verification

**Purpose**: Build, install, and verify the full feature end-to-end.

- [ ] T017 Build `jitouch/Jitouch/Jitouch.xcodeproj` in Release configuration (agent must be built first)
- [ ] T018 Build `prefpane/Jitouch.xcodeproj` and install resulting `Jitouch.prefPane`
- [ ] T019 Run full end-to-end verification per `specs/001-shell-command/quickstart.md` including cross-tab checks

---

## Dependencies & Execution Order

### Phase Dependencies

- **Foundational (Phase 1)**: No dependencies — start immediately; all three tasks [P]
- **US1/US2 (Phase 2)**: Depends on T001 (TrackpadTab.h header)
  - T004 and T005 can start in parallel (different files)
  - T006, T007, T008 are sequential within TrackpadTab.m but can start after T004
- **US3 (Phase 3)**: Depends on T002 (MagicMouseTab.h) and T003 (RecognitionTab.h);
  best started after Phase 2 so the pattern is confirmed before mirroring
- **Polish (Phase 4)**: Depends on all implementation phases complete

### User Story Dependencies

- **US1 (P1)**: Depends on Foundational (T001); core MVP
- **US2 (P2)**: Implemented within Phase 2 (T008); no separate prerequisites
- **US3 (P3)**: Mirrors Phase 2; can start after T002/T003

### Parallel Opportunities

```text
# Phase 1 — run all three simultaneously:
T001: prefpane/TrackpadTab.h
T002: prefpane/MagicMouseTab.h
T003: prefpane/RecognitionTab.h

# Phase 2 — start T004 and T005 simultaneously:
T004: prefpane/TrackpadTab.m (loadActionButton)
T005: jitouch/Jitouch/Gesture.m (doCommand)

# Phase 3 — pair by method across the two files:
T009 + T010: loadActionButton (MagicMouseTab || RecognitionTab)
T011 + T012: change: (MagicMouseTab || RecognitionTab)
T013 + T014: commitCommandSheet: (MagicMouseTab || RecognitionTab)
T015 + T016: showCommandSheet (MagicMouseTab || RecognitionTab)
```

---

## Implementation Strategy

### MVP (User Story 1 + 2 Only)

1. Complete Phase 1 (T001–T003)
2. Complete Phase 2 (T004–T008) — TrackpadTab only
3. **STOP and VALIDATE**: build both targets, assign on Trackpad tab, trigger gesture,
   verify file created, edit command, verify new command runs
4. Ship MVP if sufficient; proceed to US3 when ready

### Full Delivery

1. Phase 1 (headers) → Phase 2 (TrackpadTab + Gesture.m) → validate → Phase 3 (other two tabs) → Phase 4 (final build + verification)

---

## Notes

- T004 and T005 are in different projects (prefpane vs jitouch agent) — truly independent
- All Phase 3 tasks are mechanical mirrors of Phase 2; use Phase 2 as the reference
- `[P]` in Phase 3 refers to pairing: each MagicMouseTab task parallels its RecognitionTab counterpart
- Build order (agent before prefpane) applies at T017/T018 — do not reverse
