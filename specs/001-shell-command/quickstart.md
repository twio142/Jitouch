# Quickstart: Verifying Shell Command Support

## Prerequisites

- Xcode installed
- Jitouch source checked out on branch `001-shell-command`

## Build

1. Open `jitouch/Jitouch/Jitouch.xcodeproj` in Xcode.
   Set scheme to **Release**. Build (⌘B). Confirm `Jitouch.app` appears in `prefpane/`.

2. Open `prefpane/Jitouch.xcodeproj` in Xcode.
   Build (⌘B). Confirm `Jitouch.prefPane` is produced.

3. Double-click `Jitouch.prefPane` to install. Click **Install** when prompted.

## Assign a Shell Command Gesture

1. Open **System Preferences** → **Jitouch**.
2. Select any gesture (e.g., Three-Finger Swipe Right on the Trackpad tab).
3. Click the action dropdown. Confirm **"Run Shell Command..."** appears in the list.
4. Select **"Run Shell Command..."**.
5. In the prompt that appears, type: `touch /tmp/jitouch_test_file`
6. Click **OK**.
7. Confirm the dropdown now shows `Run "touch /tmp/jitouch_test_file"`.
8. Click **Apply** (or the equivalent save button).

## Trigger the Gesture

Perform the gesture on your trackpad. Wait 1-2 seconds.

## Verify

```bash
ls /tmp/jitouch_test_file
```

Expected: the file exists. If it does, the feature works end-to-end.

## Cleanup

```bash
rm /tmp/jitouch_test_file
```

## Cross-Tab Verification

Repeat the assign/trigger/verify steps on:
- The **Magic Mouse** tab (if a Magic Mouse is available)
- The **Recognition** tab (character recognition gestures)

Each tab must show "Run Shell Command..." in the dropdown and execute the command when
triggered.
