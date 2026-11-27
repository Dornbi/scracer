# Stunt Car Racer (C64) — Disassembly & Performance Optimization

This is a fun little, probably somewhat controversial idea:

- Take a classic C64 game.
- Use AI tools to completely disassemble it & reassemble it.
- Build a harness to preserve the functionality and make sure it is pixel by pixel identical.
- And then let an agent run loose to optimize it.

The original reassembled version is bit by bit identical to the original, and has about 200 bytes of RAM free fragmented across a dozen blocks.

<video src="https://github.com/user-attachments/assets/d646780a-2b27-4f9f-8884-b95013b27e02" controls="controls" style="max-width: 100%;"></video>

## Goal

Make the original 1989 C64 version run noticeably faster on real 6510 hardware while keeping the game 100% faithful to the original:

- **Pixel-identical rendering**: `0` differing pixels across all 10 deterministic PAL verification scenes (title, grid, boost, jumps, banking, walls, airborne wheel struts, and live opponent car).
- **Bit-exact physics & math**: `0` differing bytes in physics state RAM (`$0100–$015F`) and `686/686` on-target 6510 unit tests (19 suites) matching the unmodified original binary's digest.
- **Zero features cut**: single player, multiplayer, league progression, Hall of Fame, disk/tape save & load, SID audio, and all 8 tracks remain intact.
- **Relocatable source & standalone `.prg`**: all 4 code modules assemble cleanly without fragile hardcoded addresses (`make verify-reloc`), recompute the boot anti-tamper checksum automatically, and pack into a standalone single-file C64 `.prg`.

## Results after Pass 80

Measured with C64 CIA2 chained 32-bit hardware timers in PAL `x64sc` across 9 deterministic driving scenes (`1,915,903` -> **`1,547,662` cycles/frame**, **`-19.22%`** overall):

| Scene                | Original (cyc/f) | Optimized (cyc/f) |     Speedup | PAL FPS |
| :------------------- | ---------------: | ----------------: | ----------: | ------: |
| `02_little_grid`     |          206,799 |           163,792 |     -20.80% |    6.02 |
| `03_little_boost`    |          205,914 |           165,119 |     -19.81% |    5.97 |
| `04_stepping_jump`   |          215,297 |           174,353 |     -19.02% |    5.65 |
| `05_hump_weave`      |          241,543 |           188,416 |     -21.99% |    5.23 |
| `06_high_jump`       |          208,131 |           166,823 |     -19.85% |    5.91 |
| `07_roller_bank`     |          238,156 |           188,601 |     -20.81% |    5.22 |
| `08_drawbridge_wall` |          230,701 |           193,214 |     -16.25% |    5.10 |
| `09_high_jump_air`   |          172,360 |           143,729 |     -16.61% |    6.85 |
| `10_opp_race`        |          197,002 |           163,615 |     -16.95% |    6.02 |
| **TOTAL**            |    **1,915,903** |     **1,547,662** | **-19.22%** |       — |

The optimization workflow uses a two-phase strategy: compacting cold UI, menu, disk, and setup code to free contiguous RAM, then reinvesting that space into quarter-square multiplication tables, unrolled viewport/bonnet/span/crane/line rasterizers, fast-path clipping/projection, and zero-page-crossing loop alignment. Every pass is logged in [docs/opt-loop.md](docs/opt-loop.md).

`docs/opt-loop.md` — Detailed rules, profiling breakdown, architecture invariants, baseline numbers, and full optimization pass ledger.
