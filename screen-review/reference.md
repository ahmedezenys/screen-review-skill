# Screen review reference

## Feedback doc

Collection `feedback`, doc id = image basename without extension, non `[A-Za-z0-9_-]` → `_` (`img/03-list-empty.jpg` → `03-list-empty`).

| Field | Type | Notes |
|---|---|---|
| `file` | string | `img/NN-name.jpg` |
| `section`, `title`, `state` | string | copied from manifest |
| `note` | string | first user message (legacy) |
| `strokes` | `[{c: "#hex", p: [[x,y],…]}]` | x/y normalized 0–1 over the image |
| `messages` | `[{from: "you"\|"claude", text, at}]` | the thread; `at` ISO string |
| `reply` | string | latest Claude reply (legacy fallback) |
| `submitted_at` | ISO string | set by the page on every user send; Claude never changes it |

Docs without `messages` render from `note`/`reply`. To reply: read the doc, append to `messages`, `update` with the whole array and `if_version` from the read; on version conflict re-read and retry.

## Capture

Shrink every PNG (macOS):
```bash
sips -Z 900 -s format jpeg -s formatOptions 70 in.png --out img/NN-name.jpg
```
Linux: `convert in.png -resize 900x900\> -quality 70 img/NN-name.jpg`.

### iOS simulator
```bash
xcrun simctl list devices booted                   # udid
xcrun simctl launch <udid> <bundle-id> <launch args…>
xcrun simctl io <udid> screenshot raw.png
xcrun simctl ui <udid> appearance dark|light        # dark mode
```
Env for the app: `SIMCTL_CHILD_<VAR>=…` before `simctl launch`. Reuse an existing simulator.

### Physical iPhone
```bash
xcrun devicectl list devices                        # identifier
xcrun devicectl device install app --device <id> path/to/App.app
xcrun devicectl device process launch --device <id> <bundle-id> <args…>
```
Screenshots and taps: mobile-mcp. `mobile_list_available_devices` → device id, then `mobile_save_screenshot {device, saveTo}`, navigation with `mobile_list_elements_on_screen` + `mobile_click_on_screen_at_coordinates` (batch with `mobile_batch_commands`).

### Android
```bash
adb devices
adb shell am start -n <pkg>/<activity> --es key value
adb exec-out screencap -p > raw.png
adb shell cmd uimode night yes|no                   # dark mode
```
Or mobile-mcp as above with the adb serial as device id.

### Fallback
Maestro MCP: `list_devices` → `run` a flow ending in `takeScreenshot`, or `take_screenshot`.
