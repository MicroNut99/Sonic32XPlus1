# Vic's YATSSD code in Sonic32 - where and how it is used

**YATSSD** ("Yet Another Tilemap and Super Scaler Demo") by **Victor Luchits (viciious)**,
https://github.com/viciious/yatssd - MIT licence, see `LICENSE_YATSSD.txt` (must stay with the files).

All of it runs on the **master SH2** (`SH2C/`, built by `make` into `SH2_Master.bin`). drx's slave SH2
(PWM sound, `SH2/SH2_Slave.asm`) and the 68000 game (`Sonic32.asm`) do not use it.

## 1. Vic's files, copied unchanged

The only change: Windows line endings (CR LF) turned into LF. Checked against yatssd-master with
`cmp` after removing the CRs - identical.

| File | What it is | Used by us |
|---|---|---|
| `32x.h` | 32X register names (`MARS_SYS_COMM0`, `MARS_VDP_FBCTL`, `MARS_CRAM`, `MARS_FRAMEBUFFER` ...) | everywhere in `sonic32_master.c` |
| `types.h`, `fixed.h` | types (`tilemap_t`, `rect_t`, `drawsprcmd_t`), the `DRAWSPR_*` flags, fixed-point helpers | through `draw.h` |
| `hw_32x.c`, `hw_32x.h` | 32X set-up: `Hw32xInit()` (display mode, line tables, clears both frame buffers), `canvas_width/height/pitch` (320 x 224, pitch 384) | `Hw32xInit()` once at start; `canvas_*` for addressing the frame buffer |
| `draw.c`, `draw.h`, `draw_inc.h` | `draw_setScissor()`, `draw_clip()`, and the pixel loops that copy an 8-bit sprite into the frame buffer (normal / x flip / y flip, 8- and 16-bit destination) | `draw_setScissor()` once; the loops through `draw_sprite()` |
| `dsprite.c` | `draw_sprite()`: clips a sprite, picks the right pixel loop, calls `draw_dirtyrect()` | every Tails / tails frame |
| `font.c` | `msx[]`: an 8x8 font | the bitmaps only, for the version text |
| `sh2_fixed.s` | `IDiv`, `FixedMul`, `FixedDiv` (SH2 division unit) | linked for `dsprite.c` (only its scaling path uses it) |

**Not copied** (YATSSD parts we do not use yet): `main.c`, `crt0.s` (its own start-up and 68000
program - Sonic32 keeps drx's), `dtiles.c` (tile maps), `sound.c` (its PWM mixer), `src-md/`.

## 2. What `sonic32_master.c` provides in their place

YATSSD's `main.c` / `dtiles.c` define a few things the copied files need; we define them ourselves
(each marked `[YATSSD]` in the code):

- `int debug`, `int nodraw` (0: always draw), `int window_canvas_x/y` (0: no window offset), `tilemap_t tm`
- `draw_dirtyrect()` - in YATSSD it marks tile-map areas to redraw; ours records the rectangle each sprite
  covered in each frame buffer, so the next pass over that buffer clears exactly that area
- `main()` / `_end` - only to satisfy newlib, which `hw_32x.c`'s printf helpers pull in

## 3. The calls, in order (`sonic32_master.c`, search for `[YATSSD]`)

1. `master_main()` start: `Hw32xInit(MARS_VDP_MODE_256 | MARS_VDP_PRIO_32X, 0)` - 256-colour mode,
   both frame buffers set up and cleared
2. `draw_setScissor(0, 0, canvas_width, canvas_height)` - clip sprites to the screen
3. every frame, `draw_frame()` -> `draw_sprite(x, y, w, h, stride, pixels, flags)` for the tails, then
   Tails: `pixels` = one of the 139 frames pre-drawn by `tools/tails_frames.py`, `flags` =
   `DRAWSPR_PRECISE` (+ `DRAWSPR_HFLIP` / `DRAWSPR_VFLIP` for Tails' facing). `dsprite.c` clips, calls
   the pixel loop from `draw_inc.h`, then our `draw_dirtyrect()`
4. `version_text()` reads the glyphs from `msx[]` (font.c) and writes only the letter pixels

## 4. Ours, not Vic's (same file)

The frame loop and the 68000 <-> SH2 protocol (COMM0-COMM6), the double-buffer flip with time limit,
the palette writes in vertical blank, the R1 sky / water gradients and line fills (32X VDP Auto Fill),
the version text, and the start-up copied from drx's `SH2_Master.asm`.

## 5. Next steps that would use more of YATSSD

- `dtiles.c` + `draw_tilemap()`: whole backgrounds as 32X tile maps (needs the tile-map data in Vic's
  format - his Tiled export scripts are in `extensions/`)
- `draw_stretch_sprite()` (already compiled in, `dsprite.c`): scaled sprites for effects
- `DRAWSPR_MULTICORE` + the slave SH2: split drawing over both SH2s (the slave is busy with drx's PWM
  sound now; YATSSD's own `sound.c` could replace it)
