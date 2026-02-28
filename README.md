# godot-interact

Simulate player input in Godot 4 scenes and capture the result as PNG. Built for AI-assisted visual iteration with [Claude Code](https://claude.ai/code).

## The Problem

AI agents can capture static screenshots of Godot scenes (via [godot-preview](https://github.com/TJ-Dev-Studio/godot-preview)), but can't test what happens when a player taps a button, drags the screen, or presses a key.

## The Solution

`godot-interact` injects an input replay script into the scene, runs Godot with `--write-movie`, and extracts the result as PNG. The AI agent sees exactly what the player would see after interacting.

```bash
# Simulate a tap and see the result
./godot_interact.sh ./my_game res://scenes/lobby.tscn \
  --actions '{"actions":[{"type":"wait","seconds":2},{"type":"touch","position":[540,960]}]}'
```

## Install

```bash
git clone https://github.com/TJ-Dev-Studio/godot-interact.git
cd godot-interact
chmod +x godot_interact.sh

# Requirements
brew install ffmpeg
# Godot 4.x must be installed
```

## Usage

```bash
# Tap the center of the screen
./godot_interact.sh ./project res://scenes/level.tscn \
  --actions '{"actions":[{"type":"wait","seconds":2},{"type":"touch","position":[540,960]}]}'

# Drag across the screen (pan gesture)
./godot_interact.sh ./project res://scenes/level.tscn \
  --actions '{"actions":[{"type":"drag","from":[800,960],"to":[200,960],"duration":1}]}'

# Load actions from a file
./godot_interact.sh ./project res://scenes/level.tscn --actions actions.json
```

See [CLAUDE.md](CLAUDE.md) for the full AI agent workflow guide.

## How It Works

1. Generates a GDScript that replays your input actions via `Input.parse_input_event()`
2. Temporarily adds it as an autoload to `project.godot`
3. Launches Godot with `--write-movie` to capture the interaction
4. Extracts the final frame as PNG with ffmpeg
5. Restores `project.godot` and cleans up

No display, no screen recording, no manual testing required.

Part of the [Godot Agent Kit](https://github.com/TJ-Dev-Studio/godot-agent-kit).

## License

MIT
