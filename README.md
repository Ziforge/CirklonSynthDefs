# Sequentix Cirklon Instrument Definitions

JSON `.cki` instrument definitions for the **Sequentix Cirklon**, plus device
setup notes for a Make Noise + FH-2 rig.

**Synths (MIDI CC maps):** Plinky (synth/sampler), Korg phase8, Elektron
Digitone II · Analog Rytm MKII (+FX/Perf) · Octatrack MKII, OTO BAM/BIM/BOUM,
Moog MF-108M Cluster Flux.

**Rig (CV + MIDI):** `CVIO-Modular.cki` (DPO, Spectraphon A/B, RxMx, Morphagene),
`NUSS.cki` (MultiWAVE, 4 modes), `FH2.cki` (FH-2 expander).

## Setup docs

- [`CVIO-Setup.md`](CVIO-Setup.md) — CVIO Config, CV/gate budget, clock.
- [`NUSS-MIDI.md`](NUSS-MIDI.md) — MultiWAVE MIDI map + modes.
- [`FH2-Setup.md`](FH2-Setup.md) — FH-2 converters + CC maps.
- [`MPE-Setup.md`](MPE-Setup.md) — ERAE II MPE paths.

USB routing (laptop-central): `usb1`→FH-2, `usb2`→MultiWAVE, `usb3`→Phase 8,
`usb4`→Ableton; CVIO defs use the internal `"CV"` port.

## Use

Copy `.cki` files to the Cirklon SD card → `SETUP > INSTR DEFS > LOAD` → assign
to a track and set the port/channel to match your cabling.

MIT © George Redpath ([@Ziforge](https://github.com/Ziforge))
