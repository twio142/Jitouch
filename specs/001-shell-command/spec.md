# Feature Specification: Custom Shell Command Support

**Feature Branch**: `001-shell-command`

**Created**: 2026-05-17

**Status**: Draft

**Input**: User description: "implement @PLAN.md"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Assign Shell Command to Gesture (Priority: P1)

A user wants to trigger an arbitrary shell command by performing a multi-touch gesture.
They open System Preferences → Jitouch, select a gesture, choose "Run Shell Command..."
from the action dropdown, type a command, confirm, and save. When the gesture is
subsequently performed, the shell command executes in the background.

**Why this priority**: This is the entire feature. Without it nothing else delivers value.

**Independent Test**: Can be fully tested by assigning `touch /tmp/jitouch_test_file` to
any gesture and triggering it — success is confirmed when the file exists afterward.

**Acceptance Scenarios**:

1. **Given** the Jitouch preference pane is open, **When** the user selects a gesture and
   opens the action dropdown, **Then** "Run Shell Command..." appears as an option in the
   list alongside "Open File..." and "Open Website...".

2. **Given** the user selects "Run Shell Command...", **When** the prompt appears,
   **Then** the user can type any shell command and confirm with "OK".

3. **Given** a shell command has been assigned to a gesture, **When** the gesture is
   performed, **Then** the shell command executes without blocking the gesture system or
   producing visible UI.

4. **Given** a shell command has been assigned, **When** the user reopens the action
   dropdown for that gesture, **Then** the dropdown displays the assigned command as
   `Run "<command>"` so the user knows what is configured.

---

### User Story 2 - Edit or Replace an Existing Shell Command (Priority: P2)

A user has previously assigned a shell command to a gesture and now wants to change it.
They select the gesture, see the current command displayed in the dropdown, choose
"Run Shell Command..." again, update the command, and confirm.

**Why this priority**: Without editability, the feature is difficult to use in practice.

**Independent Test**: Assign a command, reopen the pane, change the command to something
different, trigger the gesture, verify the new command runs (not the old one).

**Acceptance Scenarios**:

1. **Given** a shell command is already assigned, **When** the user selects the gesture
   and views the action dropdown, **Then** the current command is shown as
   `Run "<command>"` so they know what is set.

2. **Given** the user selects "Run Shell Command..." again and enters a new value,
   **When** they confirm, **Then** the new command replaces the old one and is saved.

---

### User Story 3 - Consistent Behavior Across All Device Types (Priority: P3)

The shell command option is available for all three supported device types: MacBook
Trackpad, Magic Mouse, and Magic Trackpad (via the Recognition tab).

**Why this priority**: Users switching between devices should have the same capabilities
regardless of which device they are configuring.

**Independent Test**: Assign a shell command gesture on each of the three device tabs and
verify execution works for each.

**Acceptance Scenarios**:

1. **Given** the user is on any of the three device tabs (Trackpad, Magic Mouse,
   Recognition), **When** they configure a gesture action, **Then** "Run Shell Command..."
   is present in the dropdown and behaves identically across all tabs.

---

### Edge Cases

- What happens when the shell command is an empty string or only whitespace?
- What happens when the shell command launches a long-running process (e.g., `sleep 60`)?
- What happens if the shell command fails or exits with a non-zero status?
- What happens when the user dismisses the prompt without entering a command ("Cancel")?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The gesture action dropdown MUST include "Run Shell Command..." as an option
  for all three device configuration tabs.
- **FR-002**: Selecting "Run Shell Command..." MUST present a text input prompt allowing
  the user to type any shell command string.
- **FR-003**: The user MUST be able to cancel the prompt without changing the current
  gesture action.
- **FR-004**: After confirming a shell command, the assigned command MUST be persisted to
  the gesture's saved configuration.
- **FR-005**: The gesture action dropdown MUST display the currently assigned shell command
  as `Run "<command>"` when a shell command is configured for that gesture.
- **FR-006**: When a gesture with a shell command action is triggered, the system MUST
  execute the command using the user's default shell in a background process that does not
  block gesture recognition.
- **FR-007**: Shell command execution MUST NOT produce any visible UI or notification
  unless the command itself does so.
- **FR-008**: The shell command feature MUST behave identically across the Trackpad,
  Magic Mouse, and Recognition configuration tabs.

### Key Entities

- **Gesture**: A multi-touch input pattern mapped to an action.
- **Command Dictionary**: The per-gesture configuration record that stores the action type
  and its parameters (e.g., the shell command string).
- **Shell Command**: An arbitrary string executed by the system shell when the gesture
  fires.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A user can assign a shell command to a gesture and trigger it successfully
  within one configuration session (open pane → assign → trigger).
- **SC-002**: The configured shell command is preserved across Jitouch restarts and
  system reboots.
- **SC-003**: Shell command execution does not introduce perceptible delay to gesture
  recognition — the gesture completes in the same time as built-in actions.
- **SC-004**: The feature works identically on all three device configuration tabs with no
  tab requiring different steps from the user.

## Assumptions

- The shell command is executed as the currently logged-in user with their default
  environment.
- No sandboxing or security validation of the command string is performed; the user is
  responsible for what they assign.
- Long-running commands launched by the gesture do not need to be tracked or cancellable
  from within Jitouch.
- The "Cancel" action on the input prompt leaves the gesture's previous action unchanged.
- One shell command string per gesture is sufficient; multiple commands can be chained
  using shell operators (`&&`, `;`, etc.) within the single string.
