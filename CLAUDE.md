# godot-interact

Simulate player input in Godot 4 scenes and capture the result as PNG. Built for AI agents that need to see what happens when a player taps, drags, or scrolls.

## How It Works

1. Generates a GDScript autoload that replays your input actions
2. Temporarily injects it into the Godot project
3. Launches Godot with `--write-movie` to render the interaction
4. Extracts the final frame as PNG with ffmpeg
5. Cleans up — restores project.godot to its original state

## Quick Start

```bash
# Wait 2 seconds, then tap the center of the screen
./godot_interact.sh ./my_project res://scenes/lobby.tscn \
  --actions '{"actions":[{"type":"wait","seconds":2},{"type":"touch","position":[540,960]}]}'

# Read the result PNG
# Use Read tool on /tmp/godot-interact/lobby_interact.png
```

## Action Types

### `wait` — Pause before next action
```json
{"type": "wait", "seconds": 2}
```

### `touch` — Tap at viewport coordinates
```json
{"type": "touch", "position": [540, 960]}
```

### `drag` — Touch-drag between two points
```json
{"type": "drag", "from": [540, 960], "to": [200, 960], "duration": 0.5}
```

### `key` — Press a Godot input action
```json
{"type": "key", "key": "ui_accept"}
{"type": "key", "key": "ui_right", "duration": 1.0}
```

### `scroll` — Mouse wheel at position
```json
{"type": "scroll", "position": [540, 960], "direction": "up", "amount": 3}
```

## Interaction Workflow (for AI agents)

```
1. Identify what user interaction you want to test
2. Define actions JSON (tap button, drag, etc.)
3. Run: ./godot_interact.sh <project> <scene> --actions '<json>'
4. Read the output PNG to see the result
5. Adjust scene/scripts based on what you see
6. Repeat
```

### Example: Testing a touch-drag pan

```bash
# First, capture the static scene
godot-preview capture ./project res://scenes/lobby.tscn

# Then test a horizontal drag (like a user panning)
./godot_interact.sh ./project res://scenes/lobby.tscn \
  --actions '{"actions":[
    {"type":"wait","seconds":2},
    {"type":"drag","from":[800,960],"to":[200,960],"duration":1.0},
    {"type":"wait","seconds":1}
  ]}'
# Read /tmp/godot-interact/lobby_interact.png to see post-drag state
```

## Options

| Flag | Default | Description |
|------|---------|-------------|
| `--actions, -a` | (required) | JSON string or path to .json file |
| `-r, --resolution` | `1080x1920` | Viewport resolution (WxH) |
| `-f, --frame` | last | Frame number(s) to extract |
| `-s, --seconds` | `15` | Total render time |
| `--fps` | `2` | Render FPS |
| `--capture` | `last` | When to capture: "last" or frame numbers |
| `-o, --output` | `/tmp/godot-interact/<scene>_interact.png` | Output path |
| `-v, --verbose` | off | Show Godot output |

## Requirements

- Godot 4.x
- ffmpeg

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `GODOT_BIN` | `/Applications/Godot.app/Contents/MacOS/Godot` | Path to Godot binary |

## Safety

The tool temporarily modifies `project.godot` to inject the replay autoload, but **always restores it** after capture (even on error). The injected `_interact_replay.gd` file is also cleaned up.

## Tips

- Use `--fps 2` (default) or higher for smooth drag simulations
- `--seconds 15` gives enough time for most interaction sequences
- The 1-second startup delay lets the scene initialize before input begins
- Viewport coordinates match the resolution: `[540, 960]` is center of 1080x1920
- Input action names match Godot's `Input.action_*` system (e.g., `ui_accept`, `ui_up`)
