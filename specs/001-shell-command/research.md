# Research: Custom Shell Command Support

## Shell Command Execution in Objective-C

**Decision**: Use `[NSTask launchedTaskWithLaunchPath:@"/bin/sh" arguments:@[@"-c", shellCommand]]`

**Rationale**: This is the minimal, existing-pattern approach. `/bin/sh -c <cmd>` gives
the user access to their full shell syntax (pipes, redirects, variable expansion) without
needing to locate or invoke a user-configured shell. The codebase already uses this exact
form for AppleScript file execution (Gesture.m ~line 930).

**Alternatives considered**:
- `NSTask` with the user's `$SHELL`: More correct for user preference but requires
  resolving the shell path and adds complexity; not needed since `/bin/sh` handles all
  POSIX syntax.
- `system()` C call: Simpler but blocks the calling thread until completion — unacceptable
  in a gesture callback.
- `NSWorkspace openURL:`: Only appropriate for URL-scheme-based actions.

---

## Memory Management Pattern

**Decision**: Wrap the `NSTask` launch in an `NSAutoreleasePool` using
`[[NSAutoreleasePool alloc] init]` / `[pool release]`.

**Rationale**: The agent (`Gesture.m`) uses MRC (manual retain/release) with no ARC.
The existing `OpenURL` block (Gesture.m ~line 940) uses exactly this pattern. Deviating
from it would be a Constitution Principle IV violation.

**Alternatives considered**:
- Omitting the pool: Would leak autoreleased objects; not acceptable.
- Using `@autoreleasepool {}`: Correct in ARC but syntactically inconsistent with the
  rest of Gesture.m.

---

## UI Input Pattern for Shell Command Text

**Decision**: Use `NSAlert` with an `NSTextField` set as `accessoryView`, presented as
a sheet on `commandSheet`.

**Rationale**: The URL entry flow uses a dedicated `.xib` window (`urlWindow`) wired up
via Interface Builder outlets. For a single text input, this overhead is unnecessary.
`NSAlert` with `accessoryView` achieves the same result in ~10 lines of code with no IB
changes. The PLAN.md specification explicitly calls out this approach.

**Alternatives considered**:
- Reusing the URL window with relabelled text: Would require IB edits and conflates
  URL-specific logic (scheme validation) with shell command input.
- `NSTextField` in a custom `NSPanel`: More code, no user benefit.

---

## State Variable Naming

**Decision**: Use `NSString *shellCommand` as the ivar name in all three tab header files.

**Rationale**: Mirrors the naming of `openFilePath` and `openURL` in the same files.
Consistent with Constitution Principle IV (follow existing patterns).

---

## Dictionary Key Name

**Decision**: `"ExecuteShellCommand"` as specified in PLAN.md.

**Rationale**: Consistent with the existing key naming style (`"OpenFilePath"`,
`"OpenURL"`). Descriptive enough to self-document its intent when read in Settings or
debug output.

---

## Three-Tab Implementation Strategy

**Decision**: Apply identical changes to TrackpadTab, MagicMouseTab, and RecognitionTab
in a single pass.

**Rationale**: Constitution Principle III (Command Dictionary Contract) requires all three
tabs to implement the same four integration points. Constitution Development Workflow
(Three-Tab Rule) prohibits divergence. The three tab files share the same method
signatures and patterns, making the changes mechanical and parallel.
