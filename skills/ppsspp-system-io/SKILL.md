---
description: "PPSSPP system control and I/O tools: emulator status, game info, loading/unloading games, save states, controller input (buttons, sequences, analog sticks), kernel threads, HLE modules, screenshots. Use when loading or reloading a game, managing save states, automating input or menu navigation, inspecting threads, or taking screenshots."
---

# PPSSPP System Control & I/O Tools

## System Status (2 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `get_status()` | -- | Get the current emulator status (running, paused, stepping, no game loaded) |
| `get_game_info()` | -- | Get information about the currently loaded game (title, disc ID, version) |

**Notes:**
- `get_status` works even with no game loaded -- useful for checking if the emulator is ready.
- `get_status` returns: `status` (one of: "no_game", "stepping", "paused", "running"), `gameLoaded` (bool), `stepping` (bool), and `pc` (hex, only when a game is loaded).
- `get_game_info` returns: `id` (disc ID), `version` (disc version string), and `title`. Returns error if no game is loaded.

## Game Loading (2 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `load_game(path)` | path: ISO, CSO, PBP or ELF | Stop whatever is running and boot the given game |
| `unload_game()` | -- | Stop the running game and return to the menu |

**Notes:**
- `load_game` always stops the current game first, so it also reloads a file rebuilt in place at the same path.
- It waits for boot (up to 60 s) and returns `result`, `status` ("running"/"stepping"), `path`, and `title`/`disc_id` when available.

## Save States (3 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `save_state(slot?, path?)` | slot: number (default: current slot); path: explicit file | Save a state to a slot or to a file |
| `load_state(slot?, path?)` | same as `save_state` | Load a state from a slot or from a file |
| `list_save_states()` | -- | List the slots for the running game with their timestamps |

**Notes:**
- `path` wins over `slot` and bypasses the slot machinery (no undo copies), e.g. `"/tmp/options.ppst"`.
- Returns `result` ("success"/"warning"/"failure"), `path`, `slot` (for slot operations) and an optional `message`.
- Works while paused. Times out after 15 s if the emulator is not running frames or save states are blocked (netplay, achievements hardcore mode).
- `list_save_states` returns `current_slot` and a `slots` array (`slot`, `used`, and `saved`/`path` when used).

## Controller Input (6 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `press_button(button)` | button name(s) | Hold a button until `release_button` |
| `release_button(button)` | button name(s) | Release a held button |
| `tap_button(button, duration_ms?)` | duration_ms: default 120, clamped 16-5000 | Press, wait, release |
| `input_sequence(sequence, hold_ms?, gap_ms?)` | hold_ms default 120, gap_ms default 80 | Play a comma separated list of presses in one call |
| `set_analog(stick?, x?, y?)` | stick: 0 left (default) / 1 right; x, y: -1 to 1 | Set an analog stick; (0,0) recenters it |
| `get_input_state()` | -- | Held buttons (`buttons_hex`, `pressed`) and both stick positions |

**Button names:** `cross` (`x`), `circle` (`o`), `square`, `triangle`, `up`, `down`, `left`, `right`, `start`, `select`, `ltrigger` (`l`), `rtrigger` (`r`). Combine with `+`, e.g. `"ltrigger+rtrigger"`.

**Notes:**
- `tap_button` and `input_sequence` need emulation **running** -- a press only registers while frames advance. They fail if the emulator is paused.
- Sequence steps are `button`, `button:ms` or `wait:ms`, e.g. `"start, down, down, cross:200, wait:1500, circle"`. At most 64 steps and 60 seconds total; it stops early if a breakpoint pauses emulation.

## Thread Inspection (1 tool)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `list_threads()` | -- | List PSP kernel threads with their status and PC |

**Notes:**
- Returns an array of threads, each with: `name`, `id`, `status` (running/ready/waiting/dormant/dead/suspended), `pc` (hex), `entrypoint` (hex), `priority` (int), `isCurrent` (bool).
- Useful for understanding which thread is executing, finding thread entry points, and debugging multi-threaded games.
- The PSP kernel is multithreaded -- games typically have separate threads for main logic, rendering, audio, and I/O.

## HLE Modules (1 tool)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `list_hle_modules()` | -- | List HLE modules and functions imported by the running game |

**Notes:**
- Returns modules with their imports, including function name, NID, and stub address for each imported function.
- Useful for understanding which PSP OS APIs the game uses.
- Combine with `lookup_symbol` to cross-reference HLE function stubs with addresses found in disassembly.
- HLE imports are stored in the symbol map with a `zz_` prefix (e.g. `zz_sceDisplaySetFrameBuf`), but `lookup_symbol` handles this automatically.

## Screenshots (1 tool)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `take_screenshot(type?)` | type: "display" (default) or "render" | Capture a screenshot of the current PSP display as PNG |

**Screenshot types:**
- `display` (default) -- captures the final game output as shown on screen.
- `render` -- captures the in-progress render (may show partially drawn frame).

**Tips:**
- Returns the screenshot as an inline base64-encoded PNG image (not a file path).
- Use screenshots to verify game state, check UI navigation results, or document visual bugs.
- Combine with `pause()` to capture a specific frame.
- Has a 5-second timeout -- if the screenshot doesn't complete in time, it returns an error.
