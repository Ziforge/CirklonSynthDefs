# MPE setup — ERAE II with this rig

Status, sourced from the Sequentix developer (Colin Fraser) on the official
forum (thread *"MPE!"* t=2412, and *"Help with soft thru for MPE controller"*
t=5729, posts Apr 2023 → Jan 2026):

- **MPE soft-thru (play an MPE controller *through* the Cirklon to a synth): works.**
- **MPE record/playback into Cirklon patterns: not fully working yet.** Notes
  record on their per-note channels and play back, but per-note *expression*
  (poly-pressure, per-note bend) isn't reliably captured. As of the last dev
  word (verstaerker on fw 1.3bL, 2026-01-17; Colin acknowledging "channelisation
  isn't quite right yet" and that an MPE *instrument option* is still needed).

So: the Cirklon can be your **MPE master-keyboard hub**, but don't expect to
*sequence* a full MPE performance yet.

## Decision: two ways to use the ERAE II

### 1. Direct to the synth (best for expressive parts) — no Cirklon involved
Route the ERAE's MPE straight to the destination on the laptop:
- **→ MultiWAVE (N.U.S.S.):** enable MPE on the module (long-press MODE on
  Page 2 → Span window BLUE); Lower Zone, master ch 1, members ch 2–9. See
  `NUSS-MIDI.md`.
- **→ modular via FH-2:** set an FH-2 converter to **Type 2 (MPE)** — per-note
  CV + gate + **bend + pressure** to DPO / Spectraphon. See below.
- **→ Ableton:** an MPE-enabled instrument track.

Full 5-D expression, but the part is live, not sequenced.

### 2. Through the Cirklon as master keyboard (track selection, live)
The Cirklon soft-thrus the ERAE to the *selected* track's instrument, so picking
a track chooses what the ERAE plays. Expression passes through live; it is **not**
captured into patterns. Method depends on firmware:

**Firmware ≤ 1.22 (released) — port soft-thru, no channelisation**
- Set the MPE synth's Cirklon port soft-thru to **`1-xx`** (pass all channels
  through on their *original* channel — no re-channelisation).
- To capture *notes* (only): in the CK record menu, hold **SHIFT + press the
  "filter" encoder** to enable multi-channel record (`n` → `n-mc`). Notes record
  with their channel and play back on it. (Expression still not captured.)

**Firmware v1.3 beta — track "input sources" (cleaner)**
- Define a track **input source** = the ERAE's input port + channel range. A
  track can soft-thru its source permanently or only when it's the edit track,
  so one MPE controller drives the selected instrument without playing every
  track. This is the "redirectable MPE soft-thru" Colin added in 1.3.
- Recording still captures notes/channels but not full expression (as above).

> **Check your Cirklon firmware first** (Settings/boot screen). It decides which
> of the two methods you use. Update to the latest 1.3 beta if you want the
> input-sources workflow.

## ERAE II → FH-2 MPE converter (the modular MPE path)

To play the DPO / Spectraphon expressively from the ERAE with per-note bend and
pressure, build an **MPE converter** on the FH-2 (see `FH2-Setup.md` for the
config workflow / `fh2-midi` MCP):

- Converter **Type = 2 (MPE)**; **Channel** = MPE global channel (ch 1);
  voice channels start one above; set **Last channel** to 9 for a Lower-Zone
  8-voice controller like the ERAE.
- **Polyphony** = number of simultaneous voices you want (e.g. 4–8).
- Enable per voice, in output-allocation order from **Base output**: CV
  (1V/oct) + Gate, then add **Bend out** and **Pressure out** so each voice
  carries pitch, gate, per-note bend and pressure.
- **Bend range** 48 semitones to match MPE members.

Note this competes with the FH-2's role as the CVIO overflow expander
(`FH2-Setup.md`) for outputs — an MPE converter with CV+gate+bend+pressure uses
4 outputs per voice, so a 2-voice MPE setup already fills the 8 native outputs.
Decide per session whether the FH-2 is doing overflow-mod **or** MPE-to-CV (or
add an FHX expander to do both).

## Connection (USB-only, laptop-central)

The ERAE II is a USB MIDI **device** → hangs off the **laptop hub**. The laptop's
MIDI router sends it to the chosen destination (MultiWAVE, FH-2, Ableton), and/or
into the Cirklon (`usb` in) if you're using approach 2. No DIN needed.

## Bottom line

- **We have not enabled anything MPE yet** — it's device/router config, not a
  `.cki`, so nothing lives in this repo except this note.
- **Expressive playing:** ERAE → synth direct (MultiWAVE MPE / FH-2 MPE / Ableton).
- **Master-keyboard routing:** ERAE → Cirklon soft-thru (method per firmware).
- **Sequencing MPE:** not supported yet — record notes only, keep expression live.
