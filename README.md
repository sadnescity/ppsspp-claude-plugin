# PPSSPP Claude Code Plugin

Claude Code plugin for [PPSSPP](https://github.com/sadnescity/ppsspp) PSP emulator MCP integration.

Provides MCP server configuration and reference skills for PPSSPP's built-in MCP server, enabling CPU debugging, GPU/GE inspection, memory watchpoints, framebuffer dumps, texture/CLUT/depth/stencil inspection, vertex/matrix inspection, GE display list disassembly, GE breakpoints, HLE module listing, save states, game loading, controller input, and more -- 47 MCP tools for PSP reverse engineering and testing.

## Installation

### From marketplace

Add the marketplace, then install the plugin:

```shell
/plugin marketplace add sadnescity/claude-plugins
/plugin install ppsspp@sadnescity-claude-plugins
```

### From local directory

Clone the repository and load it with `--plugin-dir`:

```shell
git clone https://github.com/sadnescity/ppsspp-claude-plugin.git
claude --plugin-dir ./ppsspp-claude-plugin
```

## Prerequisites

- [PPSSPP](https://github.com/sadnescity/ppsspp) with MCP server support (v1.20.1+)
- MCP server enabled in PPSSPP settings (default port: 27077)
- **CPU core set to Interpreter** (JIT/IR JIT not supported with MCP)

## What's Included

### MCP Server Configuration (`.mcp.json`)

Connects Claude Code to PPSSPP's built-in MCP server via Streamable HTTP on `localhost:27077`.

### Skills

| Skill | Description |
|-------|-------------|
| `ppsspp-setup` | MCP server setup, configuration, and troubleshooting |
| `ppsspp-cpu-debug` | CPU debugging: registers, disassembly, assembly, breakpoints, memory watchpoints, stepping, memory read/write/search, symbol lookup |
| `ppsspp-gpu-debug` | GPU debugging: framebuffers, GE display lists, GPU state, textures, CLUT, depth/stencil buffers, vertices, matrices, GE breakpoints, HLE modules, GPU stats |
| `ppsspp-system-io` | System control: emulator status, game info, game loading, save states, controller input, threads, HLE modules, screenshots |
| `ppsspp-workflows` | Reverse engineering workflows: find RAM writers, cheat search, memory diff, ASM patch verification, GPU display list analysis, GE event debugging |

## PSP Hardware Quick Reference

- **CPU:** Allegrex (MIPS32 4K-based), 1-333 MHz (default 222 MHz), little-endian
- **Coprocessors:** COP0 (system control), COP1 (FPU), COP2 (VFPU, 3.2 GFLOPS)
- **User RAM:** 0x08800000 - 0x0A000000 (24 MB)
- **VRAM:** 0x04000000 - 0x04200000 (2 MB)
- **Scratchpad:** 0x00010000 (16 KB)
- **Cache:** 16 KB I-cache + 16 KB D-cache per core

## Upstream

PPSSPP emulator: <https://github.com/hrydgard/ppsspp>
