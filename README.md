# Sequentix Cirklon Instrument Definitions

MIDI/CV instrument definition files (`.cki`) for the **Sequentix Cirklon**, with
device-side setup notes for a Make Noise + FH-2 rig.

## Instruments

| Definition | Control | Source |
|---|---|---|
| Plinky (Synth Mode) | 65 CC | Plinky docs |
| Plinky (Sampler Mode) | 65 CC | Plinky docs |
| Korg phase8 | 22 CC | Korg MIDI chart |
| Elektron Digitone II | 73 CC | Elektron MIDI ref |
| Elektron Analog Rytm MKII | 59 CC | Rytm manual |
| Analog Rytm MKII (FX) | 28 CC | Rytm manual |
| Analog Rytm MKII (Perf) | 12 CC | Rytm manual |
| Elektron Octatrack MKII | 64 CC | Octatrack manual |
| OTO Machines BAM / BIM / BOUM | 12 / 20 / 13 CC | OTO MIDI spec |
| Moog MF-108M Cluster Flux | 22 CC (14-bit) | MF-108M manual |
| CVIO Modular (DPO×2, Spectraphon A/B, RxMx, Morphagene) | CV/gate | Make Noise manuals |
| Make Noise N.U.S.S. (MultiWAVE) | 4 modes | MultiWAVE MIDI Inlet manual |
| Expert Sleepers FH-2 | 3 insts | FH-2 manual v1.24 |

## Rig setup (Make Noise + FH-2 + N.U.S.S.)

The `.cki` files are only half the story — CV/gate scaling, MPE and clock all
live on the devices. See the setup docs:

- [`CVIO-Setup.md`](CVIO-Setup.md) — CVIO Config for the Make Noise CV voices, the
  16 CV / 8 gate budget, and clock distribution.
- [`NUSS-MIDI.md`](NUSS-MIDI.md) — MultiWAVE MIDI map and the four selectable modes.
- [`FH2-Setup.md`](FH2-Setup.md) — FH-2 as a USB CV/gate expander (converters + CC maps).
- [`MPE-Setup.md`](MPE-Setup.md) — ERAE II MPE paths (direct-to-synth vs Cirklon soft-thru).

**USB routing (laptop-central, USB + hubs).** The laptop is the master USB host;
the Cirklon's `usb1`–`usb6` device ports are routed on the laptop: `usb1`→FH-2,
`usb2`→MultiWAVE, `usb3`→Phase 8, `usb4`→Ableton clock. CVIO defs use the internal
`"CV"` port.

## Loading onto the Cirklon

1. Copy the `.cki` files to the Cirklon SD card.
2. `SETUP > INSTR DEFS > LOAD`.
3. Assign to a track, then set the MIDI port/channel to match your cabling.

Every definition ships with `usb`/`CV`/MIDI ports set for the rig above — change
them to suit your own setup.

Two flags matter:
- `no_xpose` / `no_fts` — set where note numbers select *sounds* rather than
  pitches, so transpose/force-to-scale don't retune (drums, phase8 resonators).
- `poly_spread` — spreads a chord across consecutive channels (2–16); used for
  N.U.S.S. per-voice poly.

## `.cki` format

A `.cki` file is **JSON**. `instrument_data` holds one or more named instruments;
each has its routing, the aux `track_values` page, and a `CC_defs` map. One file
can define several instruments.

```json
{
  "instrument_data": {
    "phase8": {
      "midi_port": "usb3",
      "midi_chan": 1,
      "default_note": "C 3",
      "poly_spread": "off",
      "no_xpose": true,
      "no_fts": true,
      "track_values": {
        "slot_1": { "MIDI_CC": 12, "label": "Vel1" },
        "slot_9": { "track_control": "pgm" }
      },
      "CC_defs": {
        "CC_12": { "label": "Vel1", "min_val": 0, "max_val": 127, "start_val": 0 }
      }
    }
  }
}
```

- `midi_port` — integer `1`–`5` (serial MIDI), or a string: `"CV"` (CVIO expander),
  `"usb1"`–`"usb6"` (USB device), `"hst1"`–`"hst16"` (USB host, Cirklon 2).
- `midi_chan` — `1`–`16`. On the `"CV"` port this is the CVIO channel.
- `default_note` — Cirklon naming: MIDI note 0 = `C0`, so `C 3` = MIDI 36.
- `track_values` — aux page; each slot is a CC (`MIDI_CC` + `label`) or a built-in
  `track_control` (`pgm`, `note%`, …).
- `CC_defs` — each CC's `label`, `min_val`, `max_val`, `start_val`.

**CV / gate instruments** set `"midi_port": "CV"` and a channel; the CVIO Config
on the device maps that channel to physical outputs and their scaling — see
[`CVIO-Setup.md`](CVIO-Setup.md).

## Cluster Flux 14-bit CCs

The MF-108M's continuous controls are 14-bit (MSB CC + LSB at MSB+32). The Cirklon
has no 14-bit CC mode, so `ClusterFlux.cki` spends two aux rows on one parameter
(MSB then LSB, in row order — the intended approach per Sequentix), labelled e.g.
`DlyTm` / `DlyTm~`. Delay Time = CC 12/44, Feedback 13/45, Mix 14/46, LFO Rate
15/47, LFO Amount 16/48, Output 7/39, Portamento 5/37.

## Files

```
Plinky-SynthMode.cki  Plinky-SamplerMode.cki  phase8.cki
Digitone2.cki  RytmMKII.cki  RytmMKII-FX.cki  RytmMKII-Perf.cki  OctatrackMKII.cki
BAM.cki  BIM.cki  BOUM.cki  ClusterFlux.cki
CVIO-Modular.cki  NUSS.cki  FH2.cki
CVIO-Setup.md  NUSS-MIDI.md  FH2-Setup.md  MPE-Setup.md
Plinky_instrument_defs.pdf
```

## License

MIT © George Redpath ([@Ziforge](https://github.com/Ziforge))
