# Data Model: Custom Shell Command Support

## Entities

### Command Dictionary (existing, extended)

The per-gesture `NSDictionary` stored in `NSUserDefaults`. This feature adds one new key.

| Key | Type | Description |
|-----|------|-------------|
| `"Gesture"` | NSString | Gesture identifier (existing) |
| `"Command"` | NSString | Display name of the action (existing) |
| `"IsAction"` | NSNumber (BOOL) | Whether this is a built-in action vs key shortcut (existing) |
| `"ModifierFlags"` | NSNumber | Key modifier flags (existing) |
| `"KeyCode"` | NSNumber | Key code (existing) |
| `"Enable"` | NSNumber | Whether the gesture is enabled (existing) |
| `"OpenFilePath"` | NSString | Path to file (existing, optional) |
| `"OpenURL"` | NSString | URL string (existing, optional) |
| **`"ExecuteShellCommand"`** | **NSString** | **Shell command to execute (new, optional)** |

Only one of `"OpenFilePath"`, `"OpenURL"`, or `"ExecuteShellCommand"` will be present in
a given command dictionary at a time. Each is mutually exclusive.

### Session State (per tab instance)

Temporary ivar on each tab controller, cleared when the command sheet opens or closes.

| Variable | Type | Description |
|----------|------|-------------|
| `shellCommand` | NSString * | Holds the entered command during the sheet lifecycle |
| `openFilePath` | NSString * | Existing — cleared when shellCommand is set |
| `openURL` | NSString * | Existing — cleared when shellCommand is set |

## State Transitions

```
Sheet opens (new gesture)
  → shellCommand = nil, openFilePath = nil, openURL = nil

Sheet opens (editing existing gesture with ExecuteShellCommand)
  → shellCommand = [existingDict objectForKey:@"ExecuteShellCommand"]

User selects "Run Shell Command..." from dropdown
  → NSAlert presented
  → User confirms
    → shellCommand = enteredText, openFilePath = nil, openURL = nil
    → loadActionButton called
    → menu item "Run \"<cmd>\"" selected
  → User cancels
    → no state change

commitCommandSheet: called
  → if shellCommand: [newCommand setObject:shellCommand forKey:@"ExecuteShellCommand"]
  → shellCommand = nil (cleared for next use)

Gesture fires
  → doCommand() checks [commandDict objectForKey:@"ExecuteShellCommand"]
  → NSTask launches /bin/sh -c <shellCommand>
```

## Validation Rules

- `shellCommand` is not validated for content; the user is responsible for correctness
  (per spec Assumption 2).
- Empty string or whitespace-only input: the "OK" button should be disabled or the value
  should be treated as "no command assigned" (implementation decision — not enforced by
  the data model).
