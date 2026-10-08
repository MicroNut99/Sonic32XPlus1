# Sonic the Hedgehog 32X – with Tails, 32X colours and a CD audio credits screen

A modification of **Sonic the Hedgehog 32X** (drx's 2008 port of Sonic 1 to the Sega 32X, "Plus"
version with andlabs' / IWasAPerson's PWM fix). It adds **Tails** (from Sonic 2) drawn by the 32X,
**32X colour skies** in the zones, a **menu**, a **credits screen** with plasma effects, and
**CD audio** playback through an attached Sega CD.

---

## What it is

- A **32X cartridge game** (`Sonic32.32x`). It runs on a Genesis / Mega Drive with a 32X, and in
  emulators with 32X support (tested in **Fusion 3.64** and **ares**).
- Sonic 1, unchanged in its levels, physics, bosses and special stages – the 68000 runs Sonic 1 as
  before. Everything new is **added around it**: Tails, the 32X picture layer, the menu, the credits.
- Optionally a **32XCD** game: with a **Sega CD** attached underneath (Sonic32 in the cartridge slot
  on top) and an **audio CD** in the drive, the credits screen plays the CD's tracks ("Mode 1").

## What it is not

- **Not Sonic 2 or Sonic 3.** Only Tails and his animations come from Sonic 2; the levels are Sonic 1's.
- **Not a full second character.** Tails has no life counter, cannot be hurt, cannot die, does not
  break monitors, and (when played by player 2) passes through walls – he stands on floors and slopes
  only. The camera always follows Sonic.
- **Not a Sega CD game** (it does not boot from a disc). Without a Sega CD it works exactly the same,
  only the CD player is absent.
- **Not a full 32X remake.** The Genesis still draws Sonic 1's levels; the 32X adds a layer *behind*
  them (skies) and draws Tails. The level graphics are not redrawn in 256 colours.

---

## Controls

| Where | Button | Does |
|---|---|---|
| Title screen | Start | Play (as Sonic 1) |
| Menu | Up / Down, Start | Choose a zone / act / special stage / sound select / **CREDITS** |
| Menu | **X** | Back to the title screen |
| Level | Start | Pause (as Sonic 1; Mode + Z works while paused too) |
| Level | **Mode** (6-button pad 1) | **Call Tails** – he flies in from above the screen |
| Level | **pad 2**, any button | **Player 2 takes Tails** (see below) |

The X, Y, Z and Mode buttons need a **6-button pad** (in Fusion / ares: set the pad type to 6 buttons).

### Tails

- **Following (default):** Tails runs behind Sonic, about a quarter of a second behind, copying his
  moves with his own Sonic 2 animations (walk, run, roll, push, wait with tail swish ...).
- **Helping:** when he is rolled up (he rolls when Sonic jumps) or flying, he **destroys badniks**
  he touches (explosion, animal, points and chain bonus as for Sonic; bosses are left to Sonic), and
  he **collects rings** he touches.
- **Lost:** more than 256 pixels from Tails, or out of the level, for 1.5 seconds – he **flies back**
  in from above the screen and catches up with Sonic, even at full speed (after 4 seconds he is
  simply there). **Mode** calls him the same way at any time.
- **Player 2:** any button on pad 2 hands Tails over. Left / Right walk and run, A / B / C jump; A / B /
  C again in the air **flies** (each press a flap up, slow fall). When player 2's Tails flies and
  Sonic jumps up to just below him, Tails **grabs Sonic and carries him**; A / B / C on pad 1 lets go
  with a hop. 10 seconds without pad 2: he follows Sonic again.

---

## How the 32X is used

The Genesis (68000 + VDP) still runs and draws Sonic 1. The 32X adds a **second picture layer**
(320 x 224, 256 colours from 32,768) mixed with the Genesis picture, and two SH2 processors:

| Part | Job |
|---|---|
| **Master SH2** (new, `SH2C/`, C) | Draws Tails and his tails, the skies, the plasma, the version text; double-buffered, in step with the screen |
| **Slave SH2** (drx's, `SH2/SH2_Slave.asm`) | PWM sound (music samples, SEGA voice) – unchanged except one address |
| **68000** (`Sonic32.asm` + `tails/`) | Sonic 1, plus: Tails' logic, the menu, the credits, CD audio, and sending the SH2 what to draw |

**The 32X enhancements:**

- **Tails in 32X colours.** His 139 animation frames (from Sonic 2) are pre-drawn into 8-bit pictures
  with his own 15-colour 32X palette (`tools/tails_frames.py`) – he needs no Genesis palette line, no
  Genesis sprites and no VRAM, so Sonic 1's graphics are untouched.
- **Skies in 15-bit colour.** In Green Hill the sky and the water become 64-shade gradients (the sky
  of Sonic 1's background is made see-through at the level start, `tails/tails.asm GHZ_SkyPatch`;
  the water already was). In Marble, Star Light, Spring Yard and Scrap Brain the sky is Sonic 1's
  backdrop colour, replaced by a gradient matching each zone (deep blue to lilac, night to violet,
  purple to orange twilight, smog brown to amber). The 32X layer sits **behind** the Genesis picture
  there; Tails' colours carry the 32X "through" bit, so he stays in front. The skies go black while
  Sonic 1 fades its palette. Labyrinth has no sky (its background is solid walls).
- **The credits plasma** – 19 patterns: OpenJazz's own plasma first (its formula, sine table,
  speed and colours, redrawn every frame), then 18 colour-cycling patterns with complementary or
  multi-colour palettes, each drawn once and animated by turning the 32X palette.
- **Hardware features used:** 256-colour packed-pixel mode, double frame buffers, the **overwrite
  window** (bytes of 0 are not written – Tails' empty pixels stay see-through), **Auto Fill** (the 32X
  VDP fills lines itself – the SH2 bus stays free for the slave's sound), the priority and through
  bits (32X behind or in front of the Genesis), per-frame palette changes in vertical blank.

**68000 → master SH2:** four 16-bit COMM registers, every frame (`tails/tails.asm Obj8D_Send`):

| Register | In a level | In the credits |
|---|---|---|
| COMM0 | bit 15 toggles each frame, bit 14 fading, bits 0-2 zone (Labyrinth: the backdrop colour) | frame counter, "alive" |
| COMM2 | camera y / 2, Tails' flips, on screen, sky on, bit 8 of x / y | the pattern number |
| COMM4 | Tails' frame, his tails' frame | "SV" |
| COMM6 | Tails' screen x + 64, y + 64 | "ER" |

COMM2 = `$FFFF` outside levels (special stage, ending ...): the 32X shows nothing.

---

## Vic's code (YATSSD)

**YATSSD** by **Victor Luchits (viciious)**, https://github.com/viciious/yatssd, MIT licence
(`SH2C/LICENSE_YATSSD.txt`). Full details: **`docs/VIC_YATSSD.md`**; every use in the code is
marked **`[YATSSD]`** in `SH2C/sonic32_master.c`.

- **Copied unchanged** (line endings only): `32x.h`, `types.h`, `fixed.h`, `hw_32x.c/.h`,
  `draw.c/.h`, `draw_inc.h`, `dsprite.c`, `font.c`, `sh2_fixed.s`.
- **Used for:** the 32X set-up (`Hw32xInit`: 256-colour mode, line tables, buffers), the register
  names, and **drawing every Tails frame** (`draw_sprite` with `DRAWSPR_PRECISE | DRAWSPR_OVERWRITE`
  and the flip flags: clipping, flipping, the pixel loops); its 8x8 font for the version text.
- **Provided by us** in place of his `main.c` / `dtiles.c`: a few globals and `draw_dirtyrect()`
  (remembers what was drawn, so the next pass over that buffer clears it).
- **One limit found:** his pixel loops (`do ... while (--j > 0)`, unsigned) must not get a sprite
  clipped to fewer than 4 pixel pairs – Sonic32 skips frames with less than 8 x 2 pixels on screen.
- **Sega CD audio:** Chilly Willy's **Mode 1** code from YATSSD's `src-md` (`main.c` InitCD and the
  Sub-CPU program `cd.s`) – the method Vic's D32XR uses – rewritten for asm68k in `tails/cd.asm` and
  `tails/cd_sub.asm`: find the Sega CD BIOS at `$400000`, unpack it into the Sub-CPU's memory (with
  Sonic 1's own Kosinski decoder), copy the Sub-CPU program to `$6000`, start it, raise its level 2
  interrupt every frame; commands D (disc info: track count), P (play), S (stop), Z (pause / go on).
  The header's device field is `J6C` (C = Sega CD), so emulators attach the Mega CD.

---

## Building

Needs **Windows** (drx's assemblers `exe/ASM68K.exe`, `exe/ASMSH.exe`, 32-bit, run on 64-bit
Windows) and **WSL** with an SH2 cross compiler (`sh-elf-gcc`, SGDK-style toolchain under
`/opt/toolchains/sega`) for the master SH2.

1. Double-click **`BUILD_TAILS.bat`**. It runs, and stops with a `STOP:` line if anything fails:
   1. `wsl make` in `SH2C/` → `SH2_Master.bin` (the master SH2, with Tails' frames); the version tag
      `SONIC32-MASTER-Z6R` is checked,
   2. the slave SH2's start address (written into `SH2/OBJ_Slave.inc`),
   3. `ASMSH` → `SH2_Slave.bin`,
   4. `ASM68K` → **`Sonic32.32x`**.
2. Open `Sonic32.32x` in Fusion or ares.

Do **not** use drx's original `Assemble.bat` (kept as `docs/Assemble_original_drx.bat`): it rebuilds
drx's old master SH2 over the new one.

**Generated data** (already included, rebuild only to change them – Python 3 + Pillow):

| File | Made by | From |
|---|---|---|
| `SH2C/tails_frames.bin`, `tails_pal.bin` | `tools/tails_frames.py` (via `make S2=...`) | Sonic 2's Tails art and mappings (s2disasm) |
| `tails/tails_anim.asm` | `tools/tails_anim.py` | Sonic 2's Tails animation scripts |
| `tails/ghz_sky.bin` | `tools/ghz_sky.py` | Sonic 1's Green Hill art (this source tree) |
| `tails/credits_*.bin`, `player_glyphs.bin` | `tools/credits_roll.py` | Jazz Jackrabbit's `FONTBIG.0FN` |
| `SH2C/ojsin.h`, `ojpal.h` | (Python, see the files) | OpenJazz's sine table, `MENU.000`'s palette |

`tools/s1data.py` reads Sonic 1's Nemesis, Kosinski and Enigma data and renders its layouts.

---

## Files

| Path | What |
|---|---|
| `Sonic32.asm` | drx's Sonic 1 32X source; every change marked `SONIC32` (T1, R1, M1-M5, C1-C2, Z1 ...) |
| `tails/tails.asm` | Tails (object $8D): following, player 2, flying in, hits, carrying, sending to the SH2; the 6-button pad read; Green Hill's sky patch |
| `tails/saver.asm` | the credits screen: plasma control, credits, CD player |
| `tails/cd.asm`, `cd_sub.asm` | Sega CD Mode 1 CD audio (main side, Sub-CPU program) |
| `SH2C/sonic32_master.c` | the master SH2 program |
| `SH2C/crt_master.s`, `master.ld`, `Makefile` | its start-up, memory layout, build |
| `SH2/` | drx's slave SH2 (PWM sound) |
| `tools/` | the converters (Python) |
| `docs/` | `VIC_YATSSD.md`, `CHANGELOG.txt` (every build, what and why), drx's readmes |

---

## RAM used (68000)

Besides Tails' two object slots (`$FFFFD300` Tails, `$FFFFD1C0` his tails), his state record
`$FFFFCF80-$FFFFCFFF` and Sonic 1's own position record, Sonic32 uses only `$FFFFFF90-$FFFFFF9A`
and `$FFFFFFE8` – bytes no Sonic 1 code touches (Z2 moved them there: before, they overlapped Sonic 1's
demo flag, and the demos turned into normal games).

## Known limits

- Player-2 Tails: no walls, no monitors, no damage.
- Tails is drawn in front of foreground scenery (loops, tunnels).
- In Labyrinth, Tails' colours do not change underwater; no 32X sky there.
- At power-on there can be a short pop in the sound and a brief flash (Vic's 32X set-up clearing
  the frame buffers while the slave SH2 plays the SEGA voice) – left as it is.
- CD audio has been built from the working YATSSD / D32XR method but not yet confirmed playing.

---

## Credits

- **Sonic the Hedgehog** – Sonic Team / SEGA 1991. Tails and his art and animations from Sonic 2
  (SEGA 1992).
- **Sonic the Hedgehog 32X** – drx (with Puto, Upthorn, Hivebrain); PWM fix andlabs, IWasAPerson.
- **YATSSD, D32XR 32X code** – Victor Luchits (viciious).
- **Sega CD and 32X framework, Mode 1 CD code** – Chilly Willy.
- **Plasma** – OpenJazz (Alireza Nejati, Alister Thomson and contributors).
- **Credits font** – Jazz Jackrabbit, Epic MegaGames 1994 (`FONTBIG.0FN`).
- **This modification** – Micronut99.

## Licences – read before publishing

- Vic's YATSSD files: **MIT** (keep `SH2C/LICENSE_YATSSD.txt`).
- The plasma follows OpenJazz's: **GPL-2.0** – a published ROM containing it falls under GPL-2.0.
- Sonic 1, Sonic 2's Tails art, and Jazz Jackrabbit's font are **copyrighted game data** (SEGA, Epic).
  This source tree contains them (as drx's did for Sonic 1); the Tails frames and the credits font
  would need replacing or permission for a public release.
- Chilly Willy's Mode 1 code: check its terms (it comes with YATSSD).
