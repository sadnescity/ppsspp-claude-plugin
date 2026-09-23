---
description: "PPSSPP GPU/GE debugging tools: framebuffers, GE display list listing and disassembly, GPU state, stats, vertices and matrices, texture/CLUT/depth/stencil dumps, GE breakpoints and break-on-event. Use when debugging PSP graphics, finding the draw call behind an on-screen element, or dumping textures and buffers."
---

# PPSSPP GPU & GE Debugging Tools

## Framebuffers (2 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `list_framebuffers()` | -- | List all active virtual framebuffers tracked by the GPU |
| `get_framebuffer(address)` | -- | Dump a framebuffer as a PNG image. Emulator must be paused |

**Notes:**
- `list_framebuffers` returns each framebuffer's VRAM address, dimensions, format, and depth address.
- Use `list_framebuffers` first to find valid addresses, then pass one to `get_framebuffer`.
- `get_framebuffer` requires the emulator to be paused.

## GE Display Lists (2 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `ge_list_display_lists()` | -- | List active GE (GPU) display lists with status, PC, and stall address |
| `ge_disassemble(address, count?)` | count=32 | Disassemble GE display list commands at a given address |

**Notes:**
- `ge_disassemble` returns human-readable GPU command descriptions.
- Stops early at END command.
- Use `ge_list_display_lists` to find the current display list PC, then `ge_disassemble` to inspect its commands.

## GPU State (4 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `get_gpu_state(category?)` | category="all" | Get the current GE rendering state as human-readable strings |
| `get_gpu_stats()` | -- | Get GPU rendering statistics for the current/last frame |
| `get_current_vertices()` | -- | Get transformed vertices for the current draw call. Must be paused at a GE draw command |
| `get_gpu_matrices(name?)` | name="all" | Get GPU transformation matrices |

**Categories for `get_gpu_state`:**
- `"flags"` -- rendering flags (alpha test, depth test, blend, etc.)
- `"lighting"` -- light sources and material settings
- `"texture"` -- texture state (address, format, filtering, wrapping)
- `"settings"` -- other settings (scissor, viewport, fog, etc.)
- `"all"` (default) -- all categories

**Matrix names for `get_gpu_matrices`:**
- `"world"` -- world matrix (4x3)
- `"view"` -- view matrix (4x3)
- `"projection"` -- projection matrix (4x4)
- `"texgen"` -- texture generation matrix (4x3)
- `"bone"` -- all 8 bone matrices (each 4x3)
- `"all"` (default) -- all matrices

**Notes on `get_current_vertices`:**
- Emulator must be paused at a GE draw command (use `set_ge_break_on` with `"draw"` or `"prim"` event).
- Returns the transformed vertex data for the current primitive being drawn.

## Textures & Buffers (4 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `get_current_texture(level?)` | level=0 | Dump the currently bound GPU texture as PNG. Must be paused |
| `get_current_clut()` | -- | Dump the current CLUT (palette) as PNG. Must be paused |
| `get_depth_buffer()` | -- | Dump the current depth buffer as PNG. Must be paused |
| `get_stencil_buffer()` | -- | Dump the current stencil buffer as PNG. Must be paused |

**Notes:**
- All texture/buffer dumps require the emulator to be paused.
- `get_current_texture` supports mipmap levels via the `level` parameter (default 0).
- `get_current_clut` dumps the Color Lookup Table / palette used by indexed texture formats (CLUT4/CLUT8).
- These return inline base64-encoded PNG images.

## GE Breakpoints (3 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `set_ge_breakpoint(type, value, condition?)` | -- | Set a GE (GPU) breakpoint |
| `remove_ge_breakpoint(type, value)` | -- | Remove a GE breakpoint |
| `set_ge_break_on(event, count?)` | count=1 | Break on the next occurrence of a GE event. Emulator must be running |

**Breakpoint types for `set_ge_breakpoint` / `remove_ge_breakpoint`:**
- `"address"` -- break at a display list PC address.
- `"cmd"` -- break on a specific GE command byte (0-255).
- `"texture"` -- break when a specific texture address is used.
- `"rendertarget"` -- break when a specific render target address is used.

**Parameters:**
- `value` -- address (hex with `0x` prefix or decimal) for address/texture/rendertarget, or command number 0-255 for cmd.
- `condition` -- expression that must be true for the breakpoint to trigger (address and cmd types only).

**Events for `set_ge_break_on`:**
- `"op"` -- next GE command
- `"draw"` -- next draw call
- `"tex"` -- next texture command
- `"nontex"` -- next non-texture command
- `"frame"` -- next frame boundary
- `"vsync"` -- next vsync
- `"prim"` -- next primitive draw
- `"curve"` -- next bezier/spline
- `"blocktransfer"` -- next block transfer

**Notes:**
- `set_ge_break_on` requires the emulator to be running (not paused). It sets a one-shot break on the next matching event.
- `count` skips the first N-1 occurrences before breaking (default 1 = break immediately).
- When a GE breakpoint triggers, use `ge_disassemble`, `get_gpu_state`, `get_current_texture`, etc. to inspect GPU state.

## HLE Modules (1 tool)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `list_hle_modules()` | -- | List HLE modules and functions imported by the running game |

**Notes:**
- Returns the list of PSP HLE (High-Level Emulation) modules and their imported functions.
- Useful for understanding which PSP OS APIs the game uses (sceDisplay, sceCtrl, sceGe, etc.).
- Combine with `lookup_symbol` to find the addresses of specific HLE function stubs.
