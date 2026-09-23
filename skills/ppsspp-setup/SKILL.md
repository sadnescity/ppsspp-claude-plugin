# PPSSPP MCP Server Setup Guide

## Key Configuration Points

PPSSPP includes a built-in MCP server requiring no external processes. The MCP server is available in PPSSPP v1.20.1 and later.

Enable the MCP server in PPSSPP's developer settings. The default port is 27077.

**Important:** The MCP server requires **Interpreter mode** (not JIT or IR JIT). Change the CPU core to Interpreter in Developer Tools settings before starting the server.

## Communication Protocol

The server operates via Streamable HTTP at the endpoint `POST http://localhost:27077/mcp` using JSON-RPC 2.0 format. The server is stateless and implements MCP `2026-07-28`, while still accepting `initialize`-handshake clients on `2025-11-25`, `2025-06-18`, `2025-03-26` and `2024-11-05`.

Tools carry `readOnlyHint`/`destructiveHint` annotations: inspection tools are read-only; `write_memory`, `write_register`, `assemble`, the save state loaders/savers and `load_game`/`unload_game` are marked destructive.

## Server Information

- **Server name:** ppsspp
- **Capabilities:** tools (47 tools)
- **Instructions from server:** "PPSSPP PSP Emulator MCP server. Provides tools for inspecting and controlling the emulated PSP. Memory addresses are in PSP address space (user RAM starts at 0x08800000)."

## Address Format

All tools that accept memory addresses support:
- Hex strings with `0x` prefix: `"0x08800000"`
- Decimal numbers: `143261696`

Hex format with `0x` prefix is recommended for readability.

## PSP Memory Map

| Region | Address Range | Size |
|--------|--------------|------|
| Scratchpad | 0x00010000 | 16 KB |
| VRAM | 0x04000000 - 0x04200000 | 2 MB |
| Kernel RAM | 0x08000000 - 0x08800000 | 8 MB |
| User RAM | 0x08800000 - 0x0A000000 | 24 MB (32 MB total with kernel) |

## PSP CPU (Allegrex)

- MIPS32 4K-based, dual-core, 32-bit, little-endian
- Clock: 1-333 MHz (default 222 MHz)
- COP0 (system control), COP1 (FPU), COP2 (VFPU)
- 16 KB I-cache + 16 KB D-cache per core
- Supports selected MIPS IV instructions: ext, ins, wsbw, seb, seh, rotr, clz, clo
- Custom instructions: halt (0x70000000), mfic (0x70000024), mtic (0x70000026)
- No MMU/TLB

## Tool Categories

| Category | Tools | Count |
|----------|-------|-------|
| Execution control | pause, resume, step_into | 3 |
| Registers | read_registers, write_register | 2 |
| Disassembly & assembly | disassemble, assemble | 2 |
| CPU breakpoints | set_breakpoint, remove_breakpoint, list_breakpoints | 3 |
| Memory watchpoints | set_memcheck, remove_memcheck | 2 |
| Symbol lookup | lookup_symbol | 1 |
| Memory access | read_memory, write_memory | 2 |
| Memory search | search_memory | 1 |
| System status | get_status, get_game_info | 2 |
| Game loading | load_game, unload_game | 2 |
| Save states | save_state, load_state, list_save_states | 3 |
| Controller input | press_button, release_button, tap_button, input_sequence, set_analog, get_input_state | 6 |
| Threads | list_threads | 1 |
| HLE modules | list_hle_modules | 1 |
| Screenshots | take_screenshot | 1 |
| Framebuffers | list_framebuffers, get_framebuffer | 2 |
| GE display lists | ge_list_display_lists, ge_disassemble | 2 |
| GPU state | get_gpu_state, get_gpu_stats, get_current_vertices, get_gpu_matrices | 4 |
| Textures & buffers | get_current_texture, get_current_clut, get_depth_buffer, get_stencil_buffer | 4 |
| GE breakpoints | set_ge_breakpoint, remove_ge_breakpoint, set_ge_break_on | 3 |
| **Total** | | **47** |

## Common Issues

- Ensure the MCP server is enabled in PPSSPP settings before starting
- **The CPU core must be set to Interpreter** -- JIT and IR JIT are not supported with MCP
- Confirm port 27077 is not occupied by another service
- The server only binds to localhost -- remote connections are not supported, and requests with a non-local `Origin` header get 403
- Calling a tool name the server does not know returns JSON-RPC error `-32602` (invalid params)
- Most tools need a game to be running -- load a game first if tools return errors
- Memory read/write max size is 65536 bytes per call
- GPU buffer dumps (framebuffer, texture, depth, stencil, CLUT) require the emulator to be paused
- `set_ge_break_on` requires the emulator to be **running** (not paused)
