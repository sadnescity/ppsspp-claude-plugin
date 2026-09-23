---
description: "PPSSPP reverse engineering workflows: find code that writes to RAM, cheat search for unknown values, RAM before/after comparison, verify ASM patches, thread inspection, HLE symbol-guided exploration, live memory patching, GE display list analysis, break on GPU events, GPU performance. Use when planning or executing PSP runtime debugging tasks."
---

# PPSSPP Reverse Engineering Workflows

## Workflow A: Find What Code Writes to a RAM Address

Use this when you know the address of a variable and want to find the code responsible for modifying it.

1. `pause()` -- pause emulation
2. `set_memcheck(address=0x08XXXXXX, size=4, write=true)` -- set a write watchpoint on the target address
3. `resume()` -- resume and trigger the write in-game
4. Wait for the watchpoint to hit (emulator pauses automatically)
5. `read_registers()` -- check the `pc` register to find the current instruction
6. `disassemble(address="0x...", count=16)` -- view surrounding code for context
7. `lookup_symbol(address="0x...")` -- identify the function name if available
8. `remove_memcheck(address=0x08XXXXXX, size=4)` -- clean up when done

**Tip:** Use `change=true` to only break when the value actually changes, filtering out redundant writes.

## Workflow B: Find a Game Variable by Value (Cheat Search)

Use this when you can see a value on screen (HP, money, items) but do not know its RAM address.

1. `pause()` at a known state (e.g. when HP is displayed as 100)
2. Convert the value to hex little-endian bytes. For a 32-bit value 100: `"64000000"`
3. `search_memory(hex="64000000")` -- find all locations containing that value
4. Note the candidate addresses
5. `resume()` -- change the value in-game (e.g. take damage so HP becomes 85)
6. `pause()`
7. `search_memory(hex="55000000")` -- search for the new value (85 = 0x55)
8. Compare the two result sets -- addresses present in both searches are strong candidates
9. Verify: `read_memory(address="0x...", size=4)` -- confirm the value matches
10. Test: `write_memory(address="0x...", hex="E7030000")` -- write 999 (0x3E7) to confirm it's the right variable

**Tip:** For 16-bit values (halfwords), search with 2-byte patterns. For values that might be stored as floats, convert to IEEE 754 hex first. Use `region="vram"` or `region="scratchpad"` to search other memory regions.

## Workflow C: Compare RAM Before and After an Action

Use this to understand what a game action modifies in memory.

1. `pause()` before performing the action
2. `read_memory(address="0x08800000", size=65536)` -- save a section of RAM as baseline (repeat for multiple 64KB chunks if needed)
3. `resume()` -- perform the action in-game
4. `pause()` immediately after
5. `read_memory(address="0x08800000", size=65536)` -- read the same region again
6. Compare the two hex dumps to find changed bytes

**Tip:** Focus on specific memory regions rather than scanning all 24 MB. Use `search_memory` to narrow down areas of interest first.

## Workflow D: Verify ASM Patches at Runtime

Use this after patching code to confirm patches were applied correctly in RAM.

1. Load the game in PPSSPP
2. `pause()` -- pause at a point after the patched code is loaded
3. `read_memory(address="0x...", size=N)` -- read bytes at the patched location
4. Compare the hex bytes with the expected patch bytes
5. `disassemble(address="0x...")` -- verify instruction mnemonics are correct
6. Alternatively, use `assemble()` to apply patches at runtime:
   - `assemble(address="0x08900000", instruction="li a0, 999")` -- write a new instruction directly

**Tip:** After rebuilding the ISO at the same path, `load_game(path="...")` stops the running game and boots the new build in one call.

**Tip:** For overlays loaded from disc at runtime, advance to the game state that triggers the overlay load first. Use `list_threads()` to see if the relevant module's thread is active.

## Workflow E: Inspect Thread State

Use this to debug multi-threaded games or understand the PSP kernel's thread scheduling.

1. `pause()` -- pause emulation
2. `list_threads()` -- see all kernel threads, their status, and PCs
3. `disassemble(address="0x...")` -- examine what a specific thread is executing
4. `lookup_symbol(address="0x...")` -- identify the function the thread is in
5. `read_registers()` -- inspect the current thread's register state

**Tip:** PSP games typically have dedicated threads for main game logic, rendering (GE callbacks), audio decoding, and file I/O. Identifying which thread handles what is key to effective debugging.

## Workflow F: Symbol-Guided Exploration

Use this when starting reverse engineering on an unfamiliar game.

1. Load the game and `pause()` once it reaches a stable state
2. `list_hle_modules()` -- see all imported HLE modules and functions with their stub addresses
3. `lookup_symbol(name="sceDisplaySetFrameBuf")` -- find specific HLE function stubs
4. `set_breakpoint(address="0x...")` -- break when the function is called
5. `resume()` and wait for the breakpoint to hit
6. `read_registers()` -- inspect arguments (a0-a3) to understand how the function is called
7. `disassemble(address="0x...", stop="return")` -- follow the calling function

**Common PSP HLE modules to look up:**
- `sceDisplay` -- display/framebuffer functions
- `sceCtrl` -- controller/input functions
- `sceAudio` -- audio output
- `sceIo` -- file I/O
- `sceKernel` -- kernel/thread management
- `sceGe` -- graphics engine (GE) commands

## Workflow G: Live Memory Patching for Testing

Use this for quick experimentation without rebuilding or reassembling.

1. `pause()` -- pause the game
2. Find the target address (using Workflows B or A)
3. `read_memory(address="0x...", size=4)` -- note the original value
4. `write_memory(address="0x...", hex="NEWVALUE")` -- write new data
5. `resume()` -- observe the effect in-game
6. `take_screenshot()` -- capture the result

For code patches:
1. `disassemble(address="0x...", count=4)` -- see the original instructions
2. `assemble(address="0x...", instruction="nop")` -- NOP out an instruction
3. Or `assemble(address="0x...", instruction="li v0, 1")` -- change a return value
4. `resume()` -- test the patch

**Tip:** Always note original values before patching so you can restore them. Use `write_memory` to restore original bytes if `assemble` doesn't support the reverse operation.

## Workflow H: GPU Display List Analysis

Use this to understand how a game renders its graphics.

1. `pause()` -- pause emulation
2. `ge_list_display_lists()` -- see active display lists with their PCs
3. `ge_disassemble(address="0x...")` -- inspect the display list commands
4. `get_gpu_state()` -- view the current GPU rendering state (texture, blending, etc.)
5. `get_gpu_matrices()` -- inspect world/view/projection matrices
6. `get_current_texture()` -- dump the currently bound texture
7. `get_current_clut()` -- dump the CLUT if using indexed textures
8. `list_framebuffers()` -- see all active framebuffers
9. `get_framebuffer(address="0x...")` -- dump a specific framebuffer

**Tip:** Use `get_gpu_state(category="texture")` to focus on texture state only. The texture address in GPU state corresponds to VRAM addresses you can cross-reference with `list_framebuffers`.

## Workflow I: Break on Specific GPU Events

Use this to find the draw call responsible for rendering a specific element.

1. `resume()` -- game must be running
2. `set_ge_break_on(event="draw")` -- break on the next draw call
3. When it breaks, inspect the state:
   - `get_gpu_state(category="texture")` -- what texture is bound?
   - `get_current_texture()` -- visualize the texture
   - `get_current_vertices()` -- inspect the transformed vertex data for this draw call
   - `get_gpu_matrices()` -- check transformation matrices
   - `ge_disassemble(address="0x...")` -- see the display list commands at this point
4. If this isn't the right draw call, `resume()` and `set_ge_break_on(event="draw")` again
5. Or use `set_ge_break_on(event="draw", count=N)` to skip N-1 draw calls

**For texture-specific debugging:**
1. `set_ge_breakpoint(type="texture", value="0x04XXXXXX")` -- break when a specific texture is used
2. `resume()` -- will pause at the draw call using that texture
3. `ge_disassemble(address="0x...")` -- inspect the display list context

**For render target debugging:**
1. `set_ge_breakpoint(type="rendertarget", value="0x04XXXXXX")` -- break when rendering to a specific framebuffer
2. Use with `get_framebuffer()` to see intermediate render results

## Workflow J: GPU Performance Analysis

Use this to understand rendering performance characteristics.

1. Let the game run for a few frames
2. `pause()`
3. `get_gpu_stats()` -- see draw call counts, primitive counts, and other statistics
4. `list_framebuffers()` -- see framebuffer dimensions and formats (identify wasteful render targets)
5. `get_depth_buffer()` -- visualize depth complexity
6. `get_stencil_buffer()` -- check stencil usage patterns
