# NUSS (MultiWAVE) MIDI setup — for `NUSS.cki`

Make Noise **N.U.S.S.** (New Universal Synthesizer System) is driven over MIDI,
not CV. The **MultiWAVE MIDI Inlet** is a passive expander that adds a **USB-C
MIDI input** to MultiWAVE, taking notes/CC/MPE from a **USB MIDI host**. So the
Cirklon does *not* use the CVIO for NUSS — it sends MIDI to the Inlet.

**The MIDI controls the whole N.U.S.S. system, not just the oscillator.** MIDI
Activations enter MultiWAVE and propagate through the interconnected system —
the Channel Index (ChI) output → PoliMATHS Span, the internal MultiMod bus, and
on to the QXG low-pass gates — exactly as hardware Activation gates do. **How
much of the front end responds depends on the channel** (see below): channel 1
engages the full N.U.S.S. front end (Glide, MultiMod reset, Accumulate), while
channels 2–9 bypass it for direct per-voice control.

Sources: Make Noise *MultiWAVE MIDI Inlet* manual and *MultiWAVE* manual
(makenoise-manuals.com); Cirklon manual §2 (Poly Spread).

## Connection

MultiWAVE MIDI Inlet = USB-C **device** needing a **USB MIDI host**.

This rig is **laptop-central over USB + hubs** (no DIN): the laptop is the master
USB host and everything hangs off its hub. The MultiWAVE Inlet plugs into the
laptop; a MIDI router forwards the Cirklon's output to it. `NUSS.cki` therefore
uses **`"midi_port": "usb2"`** — one of the Cirklon's class-compliant USB *device*
virtual ports (`usb1`–`usb6`) — which the laptop routes to the MultiWAVE.
(Port map: `usb1`→FH-2, `usb2`→MultiWAVE, `usb3`→Phase 8.)

- **Alternative (direct, tighter timing):** on a Cirklon 2 you could instead host
  the Inlet on the Cirklon's own USB **host** port (`hst1`), bypassing the laptop.
  Then set the port token to `hst1`. Costs the laptop its route to that device.

## MultiWAVE MIDI implementation (MPE OFF)

| Channel | Function |
|:-------:|----------|
| 1       | Notes drive **Span-mode / Round-robin** activation (play it like a poly keyboard). Global CCs. Pitch bend ±2. |
| 2–9     | Each channel activates **one specific voice** (8-voice poly), bypassing the Span "front end". Per-channel CCs. Pitch bend ±2. |
| 10      | **Global transposition** of all channels (like the Transpose input). No CHi activity. |

MPE ON (long-press MODE on Page 2 → Span window BLUE): Ch 1 = MPE master,
Ch 2–9 = members with per-channel pitch bend ±48, Ch 10–16 disabled. Use the
lower-zone / "Lower Zone", top channel 9. This is the path for playing NUSS from
the **ERAE II / Exquis** directly over USB — not the Cirklon.

### CCs (sum with the panel pot position)

| CC  | Target        | Scope                                             |
|:---:|---------------|---------------------------------------------------|
| 2   | Spread        | Global (Ch1 / Ch10 / MPE master)                  |
| 4   | Detune        | Global                                            |
| 5   | Glide         | Global                                            |
| 8   | Blend         | Global                                            |
| 16  | Osc A Wave    | Per-channel (global on Ch1, per-voice on Ch2–9)   |
| 17  | Osc A Expo    | Per-channel                                       |
| 18  | Osc B Wave    | Per-channel                                       |
| 19  | MultiMod Time | Per-channel                                       |

Note: MIDI note values are held per channel until a new value is received; there
is a reset for all channels' pitches on the module.

## The selectable NUSS modes in `NUSS.cki`

`NUSS.cki` defines four instruments so you can assign whichever mode a track
needs on the Cirklon. All target the same `usb2` port (routed to the MIDI Inlet) on
different channels, so several can run at once on different tracks.

| Instrument     | Chan  | Front end | Use for                                        |
|----------------|:-----:|:---------:|------------------------------------------------|
| NUSS Poly      | 1     | full      | play the whole system as self-patched poly     |
| NUSS Spread    | 2→9   | bypassed  | Cirklon-allocated per-voice poly (8 voices)    |
| NUSS Voice     | 2     | bypassed  | one targeted mono voice (clone for ch 3–9)     |
| NUSS Transpose | 10    | bypassed  | global transpose of all voices + global CCs    |


- **NUSS Poly** — port `usb2`, **channel 1**, `poly_spread off`. Send polyphonic
  patterns (chords) on channel 1; MultiWAVE round-robins them across its voices
  (set the module to Round mode, Span = 1). Global CCs on the values page:
  Spread/Detune/Glide/Blend + the four osc CCs. **Engages the full N.U.S.S. front
  end — Glide, MultiMod reset and Accumulate all fire**, so the whole
  interrelated system responds. Simplest, plays immediately, and the right choice
  when you want the system behaving as a self-patched whole.

- **NUSS Spread** — port `usb2`, **base channel 2**, `poly_spread 8`. The Cirklon
  spreads a polyphonic pattern across **channels 2–9**, one note per voice, for
  independent per-voice control. The four per-channel CCs (16–19) then act on
  each voice separately. `SprMod` (CC 110) selects the Cirklon's spread
  allocation mode (0–4; 0 = least-recently-used). **Bypasses the N.U.S.S. front
  end: Glide and MultiMod reset do *not* trigger, and Accumulate has no effect.**
  Use for surgical per-voice timbre/sequencing when you don't need the front-end
  behaviours.

- **NUSS Voice** — port `usb2`, **channel 2**, `poly_spread off`. A single
  monophonic line addressing **one** MultiWAVE voice directly, with its four
  per-channel CCs. Clone it and change `midi_chan` to 3–9 to sequence any other
  individual voice (e.g. a bassline on one voice while NUSS Spread handles the
  rest). Also bypasses the front end (no Glide / MultiMod reset / Accumulate).

- **NUSS Transpose** — port `usb2`, **channel 10**, `no_xpose`/`no_fts` on. Notes
  sent here **transpose all voices instantly** (like the physical Transpose
  input); it bypasses the front end and causes no ChI activity or MultiMod bus
  reset. The four global CCs (Spread/Detune/Glide/Blend) also work on this
  channel, so it doubles as a global-parameter modulation track.

### MPE mode (played from ERAE II / Exquis, not the Cirklon)

MPE is a fifth "mode", toggled **on the module** (long-press MODE on Page 2 → Span
window BLUE), not via a `.cki`. With it on, MultiWAVE reads a Lower-Zone MPE
stream on channels 2–9 (master ch 1), giving per-note pitch bend ±48 for the
ERAE II / Exquis over USB. Turn MPE **off** again before using the Cirklon's
channel-1 Poly or channel-10 Transpose defs, since MPE disables channels 10–16
and reinterprets 2–9.

> `poly_spread` is written here as the integer `8` (Cirklon accepts 2–16). If
> your firmware exports it as a string, match that; the base channel is
> `midi_chan` (2) and voices fill upward (2→9).

## Clock / sync

NUSS syncs from **MIDI clock** on the same `usb2` port. In the Cirklon MIDI port
config, set that port's `mclk send` to `norml` (see `CVIO-Setup.md` → Clock
distribution). No CVIO gate/clock is needed for NUSS — MultiMod / PoliMATHS can
be clocked internally or from a CVIO analog clock gate if you want them locked
to a specific division.

## Transposition option

To use MultiWAVE's Ch 10 global transpose from the Cirklon, add a small extra
instrument on `usb2` / channel 10 and send notes to it — those retune all voices
instantly (bypasses the front end). Left out of `NUSS.cki` to avoid accidental
retunes; add if you want live global transposition from a dedicated track.
