# CVIO Setup — device-side configuration for `CVIO-Modular.cki`

The `.cki` file only does step 1 of the two-step CV setup: it routes a track's
output to a channel on the **`CV`** port. Everything about *how* those messages
become voltages — which physical output, V/oct vs Hz/V, note range, gate
behaviour, glide, clock division — is set on the Cirklon's **CVIO Config** page
and is **not** stored in the instrument definition.

Sources: Cirklon Operation Manual v1.20 §7–8; and the Make Noise module manuals
(DPO, RxMx, Morphagene, Spectraphon) for the real CV/gate inputs below.

## The honest budget

The CVIO has **16 CV + 8 gate** outputs. The Make Noise rig exposes **far more
CV inputs than that** (DPO ~11, Spectraphon ~10, Morphagene ~7, RxMx ~5, plus
8+ gate/clock inputs). So the CVIO **cannot and should not** drive everything.

Design principle: **the Cirklon owns pitch + note-gates + the one or two most
expressive scans per voice; the modular's own modulators (MultiMod, PoliMATHS,
Maths, etc.) cover the rest.** Gates — not CVs — are the scarce resource here.

## What each module actually accepts (from the manuals)

| Module | 1V/oct pitch | Best CV scans | Gate/clock inputs |
|--------|--------------|---------------|-------------------|
| **DPO** | VCO A **and** VCO B (2×) | Fold (0–8V), Shape (0–5V), FM Bus Index (±4V), Angle (bip), Follow (0–5V) | **Strike** (8–10V) |
| **Spectraphon** | Side A **and** Side B (2×) | Slide, Focus, Partials (all bipolar, per Side) | **Clock A / Clock B** (step spectra) |
| **RxMx** | none | Channel Select (0–5V), Radiate (0–10V), Level (0–8V) | **Strike** (8V) |
| **Morphagene** | none (Vari-Speed is **not** 1V/oct) | Organize (0–5V, splice sel), Vari-Speed (±4V), Morph (0–5V) | **CLK, Play, Shift, REC** (≥2.5V) |

Corrections vs the earlier draft: Morphagene is **not** a pitched voice
(Vari-Speed ≈ +12/−26 semitones, non-tracking) — clock/gate it instead; RxMx is
a **6-channel vactrol LPG/mixer** addressed by Channel Select CV, not four
channel CVs.

## Channel map used by `CVIO-Modular.cki`

| Instrument     | CV chan | Pitched | Notes drive        |
|----------------|:-------:|:-------:|--------------------|
| DPO            | 1       | yes     | VCO B pitch        |
| DPO OscA       | 2       | yes     | VCO A pitch        |
| Spectraphon A  | 3       | yes     | Side A pitch       |
| Spectraphon B  | 4       | yes     | Side B pitch       |
| RxMx           | 5       | no      | Strike gate timing |
| Morphagene     | 6       | no      | Play/CLK gate timing |

Each instrument also carries aux-CC lanes (Fold/Shape/…, Slide/Focus/…,
Chan/Radiat/Level, Orgnz/VarSpd/Morph). **Defining a CC costs nothing** — it
only consumes a physical CV output when you actually map it in CVIO Config.

## Recommended CVIO Config — a layout that fits 16 CV + 8 gate

### CV outputs (12 of 16 used → 4 spare)

| CV out | chan | note | ctrl | ctrl# | law   | drives                         |
|:------:|:----:|:----:|:----:|:-----:|:-----:|--------------------------------|
| 1      | 1    | 100% | 0%   | —     | V/oct | DPO VCO B pitch                |
| 2      | 2    | 100% | 0%   | —     | V/oct | DPO VCO A pitch                |
| 3      | 3    | 100% | 0%   | —     | V/oct | Spectraphon A pitch            |
| 4      | 4    | 100% | 0%   | —     | V/oct | Spectraphon B pitch            |
| 5      | 1    | 0%   | 100% | 16    | V/oct | DPO Fold (0–8V)                |
| 6      | 1    | 0%   | 100% | 19    | V/oct | DPO FM Bus Index               |
| 7      | 3    | 0%   | 100% | 16    | V/oct | Spectraphon A Slide            |
| 8      | 4    | 0%   | 100% | 16    | V/oct | Spectraphon B Slide            |
| 9      | 5    | 0%   | 100% | 16    | V/oct | RxMx Channel Select (0–5V)     |
| 10     | 5    | 0%   | 100% | 18    | V/oct | RxMx Level (0–8V)              |
| 11     | 6    | 0%   | 100% | 16    | V/oct | Morphagene Organize (0–5V)     |
| 12     | 6    | 0%   | 100% | 17    | V/oct | Morphagene Vari-Speed (±4V)    |
| 13–16  | —    | —    | —    | —     | —     | **spare** (Focus, Partials, Shape, Radiate, Morph … as needed) |

### Gate outputs (all 8 used — the tight resource)

| Gate | mode  | setting  | drives                           |
|:----:|:-----:|----------|----------------------------------|
| 1    | gate  | cv-num 1 | DPO **Strike**                   |
| 2    | gate  | chan 3   | Spectraphon **Clock A**          |
| 3    | gate  | chan 4   | Spectraphon **Clock B**          |
| 4    | gate  | chan 5   | RxMx **Strike**                  |
| 5    | gate  | chan 6   | Morphagene **Play**              |
| 6    | clock | ppqn 4   | Morphagene **CLK** (1/16 gene-step) |
| 7    | clock | ppqn 1   | general 1/4 modular clock        |
| 8    | clock | rst      | reset pulse on start             |

> Gates are the bottleneck: 8 outputs vs the rig's 8+ trigger inputs. To reach
> more (Morphagene Shift/REC, etc.), set a spare **CV output to `law = gate`**
> for an extra gate, use Morphagene's **EOSG** output to self-clock, or trigger
> from the modular's own logic/clock rather than the Cirklon.

## So, how much CV is spare?

- **As laid out above: 4 CV outputs (13–16) free, and 0 gates.**
- Only **4 CVs are truly committed** (the four pitches). Outputs 5–12 are the
  recommended scans/mod — drop any you'd rather modulate internally and those
  free up too, so realistically **4–8 CV outs are available**.
- **Gates are the real constraint**, not CVs. If you need more triggers, convert
  a spare CV to `law = gate` or offload triggering to the modular.

## Clock distribution

Cirklon is master; every clock below is locked to it.

### A. MIDI clock — per OUT port (`mclk send`, §7)

`off` / `norml` (with START/STOP) / `tempo` (ticks only). Ticks even when
stopped unless toggled (SHIFT + the `mclk send` encoder).

| Destination                     | Port    | `mclk send` |
|---------------------------------|---------|-------------|
| MultiWAVE MIDI Inlet (→ N.U.S.S.) | usb2  | norml       |
| Korg Phase 8                    | usb3  | norml       |
| Ableton (USB)                   | usb4  | norml       |

### B. Analog clock — CVIO gate ports in `clock` mode (§8)

`ppqn` = pulses per quarter note (1–48); `run` = constant-high while playing;
`rst` = reset pulse at start; `type` = norm / S-trig / inverse. In the table
above, gates 6–8 are the clock outs (Morphagene CLK, a 1/4 clock, and a reset).

> **Required:** the gate `clock` outputs only tick if `mclk send` is enabled for
> the **CV port** on the MIDI port config page (§8). Set it to `norml`.

## Verify the port token

`CVIO-Modular.cki` uses `"midi_port": "CV"`, matching the on-device port label.
If your firmware exports a different token, create one CV instrument on the
Cirklon (set MIDI Port to `CV`), save/export it, and match the exact string.
