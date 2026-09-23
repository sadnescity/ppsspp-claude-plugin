# PPSSPP CPU Debugging & Memory Tools

## Execution Control (3 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `pause()` | -- | Pause emulation (break into stepping mode) |
| `resume()` | -- | Resume emulation from stepping state, or close PPSSPP's pause menu |
| `step_into()` | -- | Step one instruction (into function calls). Must be paused first |

**Notes:**
- `step_into` requires the emulator to be paused first. Use `pause()` before stepping.
- `step_into` is synchronous -- it waits up to 100ms for the step to complete, then returns the new PC (e.g. "Stepped to 0x08900004.").
- After stepping, use `read_registers()` to inspect the full register state.

## Registers (2 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `read_registers()` | -- | Read all CPU registers: GPR (r0-r31), HI, LO, and PC |
| `write_register(register, value)` | register, value | Write a value to a single CPU register |

**Register names:** zero, at, v0-v1, a0-a3, t0-t9, s0-s7, k0-k1, gp, sp, fp, ra, hi, lo, pc. Also addressable as r0-r31.

## Disassembly & Assembly (2 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `disassemble(address, count?, stop?)` | count=16 (max 256), stop="return"\|"none" | Disassemble MIPS instructions at a given address |
| `assemble(address, instruction)` | -- | Assemble a single MIPS instruction and write it to memory |

**Parameters:**
- `address` -- hex with `0x` prefix (e.g. `"0x08900000"`) or decimal.
- `stop` -- `"return"` (default): stop at `jr ra`; `"none"`: disassemble exactly `count` instructions.

**Tips:**
- Use `disassemble` with the PC from `read_registers()` to see code around the current instruction.
- `assemble` writes the encoded instruction directly to memory at the given address. Example: `assemble(address="0x08900000", instruction="addiu a0, zero, 1")`.
- The Allegrex supports standard MIPS32 plus: ext, ins, wsbw, seb, seh, rotr, clz, clo.

## Breakpoints (3 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `set_breakpoint(address, enabled?, condition?)` | enabled=true | Set a CPU execution breakpoint at the given address |
| `remove_breakpoint(address)` | -- | Remove a CPU breakpoint |
| `list_breakpoints()` | -- | List all CPU breakpoints and memory watchpoints |

**Parameters:**
- `address` -- hex with `0x` prefix or decimal.
- `condition` -- expression that must be true for the breakpoint to trigger (e.g. `"a0 == 1"` or `"[sp+0x10] != 0"`).

**Notes:**
- When a breakpoint hits, execution pauses automatically.
- Use `read_registers()` to inspect state, then `resume()` or `step_into()` to proceed.
- Breakpoints can be created in disabled state with `enabled=false` and toggled later.
- `list_breakpoints` returns all CPU breakpoints and memory watchpoints with address, enabled status, and symbol name (if available).

## Memory Watchpoints (2 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `set_memcheck(address, size, read?, write?, change?, enabled?, condition?)` | write=true | Set a memory watchpoint on an address range |
| `remove_memcheck(address, size)` | -- | Remove a memory watchpoint |

**Parameters:**
- `address` -- start address of memory range. Hex with `0x` prefix or decimal.
- `size` -- size of memory range in bytes.
- `read` -- trigger on memory reads (default false).
- `write` -- trigger on memory writes (default true).
- `change` -- trigger only on writes that change the value (default false).
- `condition` -- expression that must be true for the watchpoint to trigger.

**Notes:**
- Memory watchpoints (memchecks) break when the CPU reads from or writes to an address range.
- Use `list_breakpoints()` to see all active watchpoints alongside CPU breakpoints.
- Combine with `read_registers()` and `disassemble()` to identify the code responsible for a memory access.
- Use `change=true` to avoid breaking on redundant writes that store the same value.

## Symbol Lookup (1 tool)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `lookup_symbol(address?, name?)` | at least one required | Look up a symbol name by address, or an address by symbol name |

**Tips:**
- Use to find function names for addresses seen in disassembly or register dumps.
- Use to find the address of a known function by name (e.g. PSP HLE function names).
- Provide either `address` or `name` (at least one required).
- Address lookup returns: label, description, function_start, and function_size (when inside a known function).
- Name lookup returns: name and address.

## Memory Access (2 tools)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `read_memory(address, size)` | max size=65536 | Read bytes from PSP memory. Returns hex string. Works with RAM (0x08800000), VRAM (0x04000000), scratchpad (0x00010000) |
| `write_memory(address, hex)` | -- | Write bytes to PSP memory. Hex string (e.g. "0102AABB"). Hex must have even length |

**Address conventions:**
- All addresses accept hex with `0x` prefix (e.g. `"0x08800000"`) or decimal.
- User RAM: 0x08800000 - 0x0A000000 (24 MB)
- VRAM: 0x04000000 - 0x04200000 (2 MB)
- Scratchpad: 0x00010000 (16 KB)

## Memory Search (1 tool)

| Tool | Parameters | Description |
|------|-----------|-------------|
| `search_memory(hex, region?, start?, end?, max_results?)` | region="ram", max_results=16 (max 256) | Search PSP memory for a byte pattern |

**Parameters:**
- `hex` -- hex string pattern to search for (even length).
- `region` -- `"ram"` (default, user RAM), `"vram"`, `"scratchpad"`, or `"kernel"`. Sets default start/end bounds.
- `start` / `end` -- address overrides. Hex with `0x` prefix or decimal.
- `max_results` -- maximum results (default 16, max 256).

**Tips:**
- Use `region` to quickly select a memory range instead of specifying start/end manually.
- Use hex byte patterns, e.g. `"64000000"` to search for the 32-bit little-endian value 100 (0x64).
- Returns an array of matching addresses and a count.
