# FH-2 expansion — device-side setup for `FH2.cki`

The FH-2 (Expert Sleepers, firmware v1.24) expands the CV/gate count beyond the
CVIO. Like the CVIO it's a **two-step** setup: `FH2.cki` sends notes/CC from the
Cirklon; the **FH-2's own configuration** maps them to physical outputs. None of
that mapping lives in the `.cki`.

Source: FH-2 User Manual v1.24
(expert-sleepers.co.uk/downloads/manuals/fh2_user_manual_1.24.pdf), pp. 26–34.

## Why the FH-2, and what it drives here

The CVIO ran out of **gates**, and a few Make Noise CV inputs didn't fit. The
FH-2 has **8 freely-assignable native outputs** (→ 64 CV + 64 gate with FHX
expanders), so it absorbs the overflow. Default allocation in `FH2.cki`:

| FH-2 out | Type | Driven by | Destination                    |
|:--------:|------|-----------|--------------------------------|
| 1        | CV   | CC 102 (Direct level) | DPO Shape (0–5V)        |
| 2        | CV   | CC 103 | DPO Angle (bipolar)               |
| 3        | CV   | CC 104 | Spectraphon A Focus (bipolar)     |
| 4        | CV   | CC 105 | Spectraphon B Focus (bipolar)     |
| 5        | CV   | CC 106 | Spectraphon Partials              |
| 6        | CV   | CC 107 | RxMx Radiate (0–10V)              |
| 7        | Gate | notes on ch 2 | Morphagene Shift             |
| 8        | Gate | notes on ch 3 | Morphagene REC               |

All reassignable — the CC number is just what the FH-2 mapping listens for.

## Connection (USB-only rig)

The FH-2 has two USB sockets: **USB A = host**, **USB C = device**. Since it
can be a device, hang it off the **laptop's USB hub via USB C**; the laptop
routes the Cirklon's output to it. `FH2.cki` uses port **`usb1`** — one of the
Cirklon's class-compliant USB device virtual ports — which the laptop's MIDI
router forwards to the FH-2. (If you instead go direct, Cirklon USB device →
FH-2 USB A host, the timing is tighter but the laptop loses the Cirklon; see the
topology note in the repo discussion.)

## FH-2 configuration to build (once, saved to flash)

Build this in the browser config tool
(expert-sleepers.co.uk/webapps/fh2_config_tool.html) or via the `fh2-midi` MCP
(`upload_config` / `save_preset`), then **Save on the module** (SysEx 18H) so it
persists.

### Two gate converters (notes → gate)

| Converter | Channel | Type | CV out | Gate out | Base output |
|-----------|:-------:|------|:------:|:--------:|:-----------:|
| 1         | 2       | Mono | off    | on       | 7           |
| 2         | 3       | Mono | off    | on       | 8           |

Disabling CV out and enabling only Gate out puts a bare gate on the Base output.
So notes on ch 2 → gate on output 7 (Morphagene Shift), ch 3 → output 8 (REC).

### Six CC → Direct-level mappings (CC → CV)

Map each CC (MIDI channel 1) to the target output's **Direct level**:

| Map | Source CC (ch 1) | Target                 |
|:---:|:----------------:|------------------------|
| 1   | 102              | Output 1 Direct level  |
| 2   | 103              | Output 2 Direct level  |
| 3   | 104              | Output 3 Direct level  |
| 4   | 105              | Output 4 Direct level  |
| 5   | 106              | Output 5 Direct level  |
| 6   | 107              | Output 6 Direct level  |

CCs 102–107 are used deliberately: mapping a CC in the **0–31** range makes the
FH-2 auto-claim CC+32 as a 14-bit LSB — using ≥102 keeps these clean 7-bit
(128-step) controls. Bump to 14-bit later by mapping a 0–31 CC pair if you want
finer resolution on a scan.

### Per-output voltage range

Set each output's range to suit its destination (Settings → output range):

- Outputs 1, 6 → **0–10V** (unipolar: DPO Shape, RxMx Radiate)
- Outputs 2–5 → **±5V** (bipolar: Angle, Focus A/B, Partials)
- Outputs 7–8 → gate levels default (5V high)

## Cirklon side (`FH2.cki`)

- **FH2 Mod** (usb1, ch 1) — six aux-CC lanes (CC 102–107) that become the six
  modulation CVs above. `no_xpose` (it's modulation, not pitch).
- **FH2 Gate A** (usb1, ch 2) — notes fire the Morphagene Shift gate.
- **FH2 Gate B** (usb1, ch 3) — notes fire the Morphagene REC gate.

## Notes / gotchas

- **Avoid CC 120 and 123** — the FH-2 treats them as All Notes Off.
- **No NRPN** — the FH-2's only mapping source is CC; build around notes + CC.
- The FH-2 has **one LFO per output** with MIDI-clock-syncable rate; you can map
  a Cirklon CC to LFO Speed/depth, or reset an LFO from a note/clock, if you'd
  rather an output self-animate than be stepped.
- Config vs preset: the **configuration** (what outputs do + CC maps) and the
  **preset** (live Direct levels / LFO rates) save independently; 30 config
  slots. Set auto-load in Settings so the rig comes up ready.
