# Sequentix Cirklon Instrument Definitions

JSON `.cki` instrument definitions for the **Sequentix Cirklon**, plus device
setup notes for a Make Noise + FH-2 rig.

**Synths (MIDI CC maps):** Plinky (synth/sampler), Korg phase8, Elektron
Digitone II (+4 machines, send FX) · Analog Rytm MKII (+FX/Perf, +32
machines) · Octatrack MKII ·
Analog Heat,
OTO BAM/BIM/BOUM, Moog MF-108M Cluster Flux.

**Rig (CV + MIDI):** `CVIO-Modular.cki` (DPO, Spectraphon A/B, RxMx, Morphagene),
`NUSS.cki` (MultiWAVE, 4 modes), `FH2.cki` (FH-2 expander).

## Setup docs

- [`CVIO-Setup.md`](CVIO-Setup.md) — CVIO Config, CV/gate budget, clock.
- [`NUSS-MIDI.md`](NUSS-MIDI.md) — MultiWAVE MIDI map + modes.
- [`FH2-Setup.md`](FH2-Setup.md) — FH-2 converters + CC maps.
- [`MPE-Setup.md`](MPE-Setup.md) — ERAE II MPE paths.

USB routing (laptop-central): `usb1`→FH-2, `usb2`→MultiWAVE, `usb3`→Phase 8,
`usb4`→Ableton; CVIO defs use the internal `"CV"` port.

## Generated definitions

`AnalogHeat.cki` and the 13 `RytmMKII-<family>.cki` files are compiled from
the parameter maps behind [`heat-mcp`](https://github.com/Ziforge/heat-mcp)
and `elektron-mcp` by `rig_midi.cirklon`, rather than written by hand. The same map drives the MCP server, so the
Cirklon's track page and the software control cannot drift apart — a test in
each device repo regenerates the `CC_defs` here and fails on any
disagreement.

That check also covers the hand-written `BAM`, `BIM`, `BOUM`, `ClusterFlux`
and `OctatrackMKII` definitions, which it reproduces exactly.

Two constraints the exporter enforces, learned from the files already here:

- Labels are **6 characters maximum** — nothing in these 500-odd CC
  definitions exceeds it.
- `CC_defs` ranges are **raw CC**, so a bipolar parameter is `0–127` with
  `start_val` 64, not its panel range.

### Digitone II machines

The Digitone needs more than one definition for two separate reasons. Its
four SYN machines share CC 40–77 and only the meaning changes — Appendix C.3
calls them "Data entry knob A–H (machine dependent)" — and its six filter
machines do the same with CC 16–24. So there is one definition per SYN
machine, each with the multi-mode filter:

| file | instrument |
|---|---|
| `Digitone2-FMTone.cki` | `DN2 FM Tone` |
| `Digitone2-FMDrum.cki` | `DN2 FM Drum` |
| `Digitone2-Wavetone.cki` | `DN2 Wavetone` |
| `Digitone2-Swarmer.cki` | `DN2 Swarmer` |
| `Digitone2-SendFX.cki` | `DN2 Send FX` — **MIDI channel 9** |

Each machine definition leads its track page with that machine's own SYN
page 1, so `DN2 FM Drum` reads `TUNE STIM SDEP ALGO OP.C OP.AB FDBK FOLD`
exactly as the device does.

**`Digitone2.cki` is still here and is not superseded.** It labels the same
CCs generically — `P1A`–`P1H` for SYN page 1 knobs A–H, as Appendix C.3
names them — so one definition works whatever machine a track runs. Use it
when you switch machines often and want the slots to stay put; use the
per-machine files when you want the knobs named. They describe the same
CCs, so comparing one against the other reports dozens of differences that
are not errors.

`Digitone2-SendFX.cki` is separate because the send effects, mixer and
master overdrive answer on the **FX CONTROL CH** rather than a track's —
which is why their CC numbers are free to repeat the track's, and they do:
the chorus and the machines both claim CC 70. Set that file's channel to
match `SETTINGS > MIDI CONFIG > CHANNELS > FX CONTROL CH`.

Labels are the device's own screen names from Appendix A, with a one-letter
section prefix added only where two sections would otherwise show the same
word — `D.MIX`, `R.MIX`, `C.MIX` for the three send mixes, rather than three
rows all reading `MIX`.

### Rytm machines

The Analog Rytm MKII needs more than one definition. Its eight SYNTH CCs,
16–23, mean something different on every machine — the manual is explicit
that the MIDI order and the SRC page order differ — and a `.cki` holds one
label per CC. So there is a definition per machine, 32 in all, grouped by
track family:

| file | machines |
|---|---|
| `RytmMKII-BD.cki` | Plastic, Sharp, Hard, Classic, FM, Silky, Acoustic |
| `RytmMKII-SD.cki` | Natural, Hard, Classic, FM, Acoustic |
| `RytmMKII-SY.cki` | Dual VCO, Chip, Raw |
| `RytmMKII-CY.cki` | Metallic, Classic, Ride |
| `RytmMKII-RS/CH/OH/HH/UT.cki` | two each |
| `RytmMKII-CP/BT/CB/Tom.cki` | one each |

Each is a complete drum-track definition — 59 CC defs, trig through LFO —
with that machine's own names on CC 16–23, so `Rytm BD Hard` leads its track
page with `LEV TUN DEC HLD SWT SWD WAV TIC`, reading as the device does.
Parameter labels are the device's own screen abbreviations, taken from
Appendix D of the manual; none is invented.

`RytmMKII.cki`, `-FX` and `-Perf` stay as they are. Those three exist
because the drum tracks, FX block and performance macros share 26 CC
numbers between them and so must live on separate MIDI channels.

## Use

Copy `.cki` files to the Cirklon SD card → `SETUP > INSTR DEFS > LOAD` → assign
to a track and set the port/channel to match your cabling.

MIT © George Redpath ([@Ziforge](https://github.com/Ziforge))
