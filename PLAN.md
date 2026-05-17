# Implementation Plan: Custom Shell Command Support

## Objective

Add the ability to map trackpad/mouse gestures to custom shell commands in Jitouch.

## Key Files & Context

- `prefpane/TrackpadTab.[h|m]`
- `prefpane/MagicMouseTab.[h|m]`
- `prefpane/RecognitionTab.[h|m]`
- `jitouch/Jitouch/Gesture.m`

## Implementation Steps

### Step 1: Update Preference Pane Headers

Add the necessary state variable for storing the shell command in the active session.

- **Files**: `prefpane/TrackpadTab.h`, `prefpane/MagicMouseTab.h`, `prefpane/RecognitionTab.h`
- **Change**: Add `NSString *shellCommand;` to the interface declarations.

### Step 2: Update Action Menus

Add the "Run Shell Command..." option to the gesture action dropdown.

- **Files**: `prefpane/TrackpadTab.m`, `prefpane/MagicMouseTab.m`, `prefpane/RecognitionTab.m`
- **Method**: `loadActionButton`
- **Change**:
    - Insert `"Run Shell Command..."` into the `builtinCommands` array (underneath `"Open Website..."`).
    - Add logic to visually display the currently assigned command in the menu:

    ```objc
    if (shellCommand) {
        [builtinCommands insertObject:[NSString stringWithFormat:@"Run \"%@\"", shellCommand] atIndex:[builtinCommands count]];
    }
    ```

### Step 3: Implement User Input (NSAlert)

Prompt the user to enter their custom shell command.

- **Files**: `prefpane/TrackpadTab.m`, `prefpane/MagicMouseTab.m`, `prefpane/RecognitionTab.m`
- **Method**: `change:`
- **Change**:
    - Add a condition for `[[actionButton titleOfSelectedItem] isEqualToString:@"Run Shell Command..."]`.
    - Create an `NSAlert` with a custom `NSTextField` as its `accessoryView`.
    - Present the alert as a sheet attached to the `commandSheet` window.
    - On "OK", save the value to `shellCommand`, clear `openFilePath`/`openURL`, and reload the action button.

### Step 4: Save the Command

Ensure the shell command is correctly saved to the gesture's configuration dictionary.

- **Files**: `prefpane/TrackpadTab.m`, `prefpane/MagicMouseTab.m`, `prefpane/RecognitionTab.m`
- **Method**: `commitCommandSheet:`
- **Change**:
    - Check if `shellCommand` is set when building `newCommand`.
    - Add `[newCommand setObject:shellCommand forKey:@"ExecuteShellCommand"];`.
    - (Also ensure that when closing/committing, we clear `shellCommand` after use where appropriate).

### Step 5: Execute the Command (Background Agent)

Handle the gesture event and execute the configured shell command.

- **Files**: `jitouch/Jitouch/Gesture.m`
- **Method**: `doCommand`
- **Change**:
    - In the execution `if/else` chain, add a check:

    ```objc
    else if ([commandDict objectForKey:@"ExecuteShellCommand"]) {
        NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
        NSString *shellCommand = [commandDict objectForKey:@"ExecuteShellCommand"];
        NSArray *shArgs = [NSArray arrayWithObjects:@"-c", shellCommand, nil];
        [NSTask launchedTaskWithLaunchPath:@"/bin/sh" arguments:shArgs];
        [pool release];
    }
    ```

## Verification & Testing

1. Build the Preference Pane and the Agent.
2. Open System Preferences -> Jitouch.
3. Select a gesture, choose action -> "Run Shell Command...".
4. Enter a harmless test command (e.g., `touch /tmp/jitouch_test_file`).
5. Ensure the action list shows `Run "touch /tmp/jitouch_test_file"`.
6. Apply changes and trigger the gesture.
7. Verify that `/tmp/jitouch_test_file` is successfully created.
