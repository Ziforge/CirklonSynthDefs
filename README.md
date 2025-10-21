# Sequentix Cirklon Instrument Definitions Collection

[![MIDI](https://img.shields.io/badge/MIDI-CC_Mapping-blue.svg)](https://midi.org)
[![Cirklon](https://img.shields.io/badge/Sequentix-Cirklon-orange.svg)](https://sequentix.com)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## Abstract

This repository provides a **collection of MIDI Control Change (CC) mapping files** (.cki format) for the **Sequentix Cirklon** hardware sequencer to control various hardware synthesizers. The instrument definitions enable comprehensive parameter automation across multiple synthesis engines, leveraging the standard MIDI 1.0 protocol for real-time performance control and studio sequencing workflows.

**Current Instruments:**
- **Plinky Synthesizer** (Synth Mode, Sampler Mode) — Touch-plate synthesizer with granular sampling, dual envelopes, four LFOs, arpeggiator, step sequencer, and effects (delay, reverb)

**Key Features:**
- Complete MIDI CC mapping for synthesis parameters (60+ per instrument)
- MPE-style multi-column position and pressure outputs (Plinky)
- Euclidean rhythm integration (arpeggiator and sequencer)
- Granular synthesis parameter control (grain size, scrub, jitter)
- Per-step parameter locks (p-locks) support

**Application**: Live electronic music performance, studio MIDI sequencing, generative music systems, hardware synthesizer integration.

---

## 1. Introduction

### 1.1 Sequentix Cirklon

The **Sequentix Cirklon** is a 16-track hardware MIDI sequencer featuring:
- **192 PPQN** (pulses per quarter note) resolution
- **Pattern-based sequencing** with up to 128 steps per pattern
- **Per-step parameter locks** (p-locks) for granular automation
- **Euclidean rhythm generator** for algorithmic pattern creation
- **MIDI CC automation** with 14-bit high-resolution control

### 1.2 Plinky Synthesizer

**Plinky** (designed by Plinkysynth) is a touch-sensitive synthesizer with:
- **8-column hexagonal touch plate** (MPE-style multi-touch)
- **Dual oscillator** (wavetable + distortion)
- **Granular sampler** (grain size, scrub, time-stretch)
- **Dual ADSR envelopes** (Env1 for amplitude, Env2 for modulation)
- **Four LFOs** (A, B, X, Y) with offset and depth control
- **Built-in arpeggiator and step sequencer** (Euclidean patterns)
- **Effects**: Delay (ping-pong, wobble), Reverb (shimmer, wobble)

### 1.3 MIDI Control Change Protocol

**MIDI CC** messages (Control Change, status byte `0xBn`) transmit parameter automation:

$$
\text{MIDI CC Message} = \langle \text{Status}, \text{CC Number}, \text{Value} \rangle
$$

where:
- **Status**: `0xB0`–`0xBF` (channel 1–16)
- **CC Number**: 0–127 (parameter ID)
- **Value**: 0–127 (7-bit resolution)

**Example** (Set Filter Resonance to maximum on channel 1):
```
0xB0 0x47 0x7F
(Channel 1, CC 71, Value 127)
```

---

## 2. Mathematical Foundations

### 2.1 MIDI-to-Parameter Scaling

**Linear scaling** from MIDI value to synthesis parameter range:

$$
P = P_{\text{min}} + \frac{V_{\text{MIDI}}}{127} \cdot (P_{\text{max}} - P_{\text{min}})
$$

where:
- $V_{\text{MIDI}} \in [0, 127]$ = MIDI CC value
- $P$ = synthesis parameter (e.g., cutoff frequency, LFO rate)
- $[P_{\text{min}}, P_{\text{max}}]$ = parameter range

**Example** (LFO Rate, 0.01–20 Hz):

$$
f_{\text{LFO}} = 0.01 + \frac{V_{\text{MIDI}}}{127} \cdot (20 - 0.01) \approx 0.01 + 0.157 \cdot V_{\text{MIDI}}
$$

### 2.2 Euclidean Rhythm Generation

**Euclidean patterns** (Toussaint, 2005) distribute $k$ pulses across $n$ steps with maximal evenness:

$$
\text{Pattern} = E(k, n)
$$

**Examples**:
- $E(3, 8) = [1, 0, 0, 1, 0, 0, 1, 0]$ (standard tresillo rhythm)
- $E(5, 8) = [1, 0, 1, 1, 0, 1, 1, 0]$ (Cuban cinquillo)

**MIDI CC control**:
- **CC 106** (Arp EuclidLen): Sets $n$ (pattern length, 1–16)
- **CC 111** (Seq EuclidLen): Sets $n$ for step sequencer

### 2.3 Granular Synthesis Parameters

**Grain synthesis** (Roads, 2004) segments audio into short overlapping grains:

$$
y(t) = \sum_{i} a_i \cdot w(t - t_i) \cdot s(t - t_i + p_i)
$$

where:
- $a_i$ = grain amplitude
- $w(t)$ = window function (Hann, Gaussian)
- $s(t)$ = source audio
- $p_i$ = grain scrub position

**Plinky grain parameters**:
- **CC 16** (GrainSize): Duration $\tau \in [1, 500]$ ms
- **CC 15** (Scrub): Position $p \in [0, 1]$ (normalized)
- **CC 17** (PlaySpeed): Playback rate $r \in [-2, +2]$ (octaves)
- **CC 18** (Timestretch): Time-stretch factor (preserves pitch)

### 2.4 ADSR Envelope Equations

**Attack-Decay-Sustain-Release** (ADSR) envelope:

$$
y(t) = \begin{cases}
\frac{t}{T_A} & 0 \leq t < T_A \\
1 - \frac{t - T_A}{T_D}(1 - S) & T_A \leq t < T_A + T_D \\
S & T_A + T_D \leq t < T_{\text{release}} \\
S \cdot e^{-\frac{t - T_{\text{release}}}{T_R}} & t \geq T_{\text{release}}
\end{cases}
$$

where:
- $T_A$ = Attack time (CC 73, Env1 Attack)
- $T_D$ = Decay time (CC 75, Env1 Decay)
- $S$ = Sustain level (CC 74, Env1 Sustain)
- $T_R$ = Release time (CC 72, Env1 Release)

---

## 3. Instrument Definition Format (.cki)

### 3.1 File Structure

```ini
[INSTRUMENT]
name = Plinky - Synth Mode
port = 1               # MIDI port (1-4)
channel = 1            # MIDI channel (1-16)
flags = poly_at        # Polyphonic aftertouch

[CC]
# CC_Number = Parameter_Name
13 = Osc Shape
71 = Resonance
...

[OUTPUT]
# MIDI CC outputs from Plinky touch columns
32 = Pos Col1
...

[PROGRAMS]
# Program change mapping
0 = Patch 1
127 = Patch 128
```

### 3.2 CC Number Allocation

**Standard MIDI CC assignments** (MIDI 1.0 spec):
- **0–31**: High-resolution controllers (14-bit with LSB at +32)
- **64–69**: Switches (sustain, portamento, etc.)
- **70–79**: Sound controllers (filter, envelope)
- **91–95**: Effects depth (reverb, chorus, delay)
- **102–119**: Undefined (available for custom use)

**Plinky custom CC assignments** (102–119):
- 102–107: Arpeggiator controls
- 108–111: Sequencer controls
- 112–114: Effect modulation (ping-pong, wobble)
- 116–118: Granular jitter parameters

---

## 4. Parameter Categories

### 4.1 Oscillator Section

| CC | Parameter | Range | Description |
|----|-----------|-------|-------------|
| 13 | Osc Shape | 0–127 | Wavetable position (sine → saw → square) |
| 4 | Osc Distortion | 0–127 | Waveshaping amount (soft → hard clip) |
| 9 | Osc Pitch | 0–127 | Coarse tuning (-24 to +24 semitones) |
| 14 | Osc Interval | 0–127 | Dual oscillator detuning (cents) |
| 2 | Noise Level | 0–127 | White noise mix (0% to 100%) |
| 5 | Glide | 0–127 | Portamento time (0 to 5 seconds) |

### 4.2 Filter and Envelopes

| CC | Parameter | Range | Description |
|----|-----------|-------|-------------|
| 71 | Resonance | 0–127 | Filter resonance (Q factor) |
| 31 | HPF Amount | 0–127 | High-pass filter cutoff |
| 73 | Env1 Attack | 0–127 | Amplitude envelope attack (1 ms to 10 s) |
| 74 | Env1 Sustain | 0–127 | Sustain level (0% to 100%) |
| 75 | Env1 Decay | 0–127 | Decay time (1 ms to 10 s) |
| 72 | Env1 Release | 0–127 | Release time (1 ms to 10 s) |
| 19–23 | Env2 Level/ADSR | 0–127 | Modulation envelope (filter, pitch) |

### 4.3 LFOs (A, B, X, Y)

| CC | Parameter | Range | Description |
|----|-----------|-------|-------------|
| 24 | LFO A Rate | 0–127 | Frequency (0.01 Hz to 40 Hz) |
| 25 | LFO A Depth | 0–127 | Modulation amount (0% to 100%) |
| 26 | LFO A Offset | 0–127 | DC offset (-50% to +50%) |
| 27–29 | LFO B Rate/Depth/Offset | 0–127 | Secondary LFO |
| 76–81 | LFO X/Y Rate/Depth/Offset | 0–127 | Touch column LFOs |

### 4.4 Arpeggiator and Sequencer

| CC | Parameter | Range | Description |
|----|-----------|-------|-------------|
| 101 | Arp Latch | 0/127 | Latch notes (on/off) |
| 102 | Arp OnOff | 0/127 | Arpeggiator enable |
| 103 | Arp Order | 0–127 | Up, down, random, as-played |
| 104 | Arp ClockDiv | 0–127 | Clock division (1/4, 1/8, 1/16 notes) |
| 105 | Arp Chance | 0–127 | Note probability (0% to 100%) |
| 106 | Arp EuclidLen | 1–16 | Euclidean pattern length |
| 107 | Arp Octaves | 1–4 | Octave range |
| 108–111 | Seq Order/ClockDiv/Chance/EuclidLen | 0–127 | Step sequencer controls |
| 83 | Seq Pattern | 0–127 | Pattern selection |
| 85 | Seq Steps | 1–16 | Sequence length |

### 4.5 Effects (Delay and Reverb)

| CC | Parameter | Range | Description |
|----|-----------|-------|-------------|
| 94 | Delay Send | 0–127 | Delay send level (0% to 100%) |
| 12 | Delay Time | 0–127 | Delay time (1 ms to 2 seconds) |
| 112 | Delay PingPong | 0/127 | Stereo ping-pong enable |
| 113 | Delay Wobble | 0–127 | Pitch modulation depth |
| 95 | Delay Feedback | 0–127 | Feedback amount (0% to 100%) |
| 91 | Reverb Send | 0–127 | Reverb send level |
| 92 | Reverb Time | 0–127 | Decay time (0.1 to 10 seconds) |
| 93 | Reverb Shimmer | 0–127 | Pitch-shift feedback (octave up) |
| 114 | Reverb Wobble | 0–127 | Reverb modulation depth |

### 4.6 Granular Sampler

| CC | Parameter | Range | Description |
|----|-----------|-------|-------------|
| 15 | Sample Scrub | 0–127 | Playhead position (0% to 100%) |
| 16 | Sample GrainSize | 0–127 | Grain duration (1 to 500 ms) |
| 17 | Sample PlaySpeed | 0–127 | Playback rate (-2 to +2 octaves) |
| 18 | Sample Timestretch | 0–127 | Pitch-independent time-stretch |
| 82 | Sample Select | 0–127 | Sample bank selection |
| 116 | Jitter Position | 0–127 | Grain start randomization |
| 117 | Jitter Grain | 0–127 | Grain size randomization |
| 118 | Jitter Rate | 0–127 | Grain spawn rate randomization |

---

## 5. MPE-Style Touch Output

### 5.1 Multi-Column Position and Pressure

**Plinky outputs MIDI CC** for 8 touch columns:

| CC Range | Function |
|----------|----------|
| 32–39 | Position Col 1–8 (X-axis touch position) |
| 40–47 | Pressure Col 1–8 (Z-axis touch pressure) |

**MPE (MIDI Polyphonic Expression)** mapping:

$$
\text{Position}_i = \frac{x_i - x_{\text{min}}}{x_{\text{max}} - x_{\text{min}}} \cdot 127
$$

where $x_i$ = touch coordinate on column $i$.

**Cirklon integration**:
- Record position/pressure CC automation per step
- Create dynamic filter sweeps, pitch bends, or LFO modulation
- Humanize sequences with pressure velocity scaling

---

## 6. Project Structure

```
CirklonSynthDefs/
├── Plinky-SynthMode.cki               # Synth mode mapping
├── Plinky-SamplerMode.cki             # Sampler mode mapping
├── Plinky_instrument_defs.pdf         # Official parameter documentation
└── README.md
```

---

## 7. Installation and Usage

### 7.1 Prerequisites

**Hardware**:
- Sequentix Cirklon sequencer (firmware 1.70+)
- Plinky synthesizer (firmware 1.07+)
- MIDI cables (5-pin DIN or USB MIDI)

### 7.2 Loading Instrument Definitions

1. **Transfer .cki files to Cirklon**:
   - Copy files to Cirklon SD card via USB
   - Navigate to `SETUP > INSTR DEFS > LOAD`

2. **Assign to track**:
   - Press `TRACK` button
   - Select `INSTRUMENT > Plinky - Synth Mode`

3. **Configure MIDI routing**:
   - Set MIDI port (1–4) and channel (1–16)
   - Enable polyphonic aftertouch if using MPE

### 7.3 Workflow Examples

**Example 1: Euclidean Arpeggiator Pattern**
```
Track 1: Plinky - Synth Mode
Step 1:  Note C4, CC 106 = 8 (EuclidLen), CC 105 = 100 (Chance 100%)
Step 2:  CC 104 = 32 (ClockDiv 1/16)
```

**Example 2: Granular Sweep Automation**
```
Track 2: Plinky - Sampler Mode
Step 1-16: CC 15 ramp 0 → 127 (Scrub position sweep)
Step 8:    CC 16 = 64 (GrainSize medium)
```

---

## 8. References

### MIDI Protocol

- MIDI Manufacturers Association (1996). *The Complete MIDI 1.0 Detailed Specification*. MMA.
- Wright, M. (2005). "Open Sound Control: State of the art 2003". *Contemporary Music Review*, 24(2-3), 91-98.

### Synthesis Techniques

- Roads, C. (2004). *Microsound*. MIT Press.
- Toussaint, G. T. (2005). "The Euclidean Algorithm Generates Traditional Musical Rhythms". *Proceedings of BRIDGES: Mathematical Connections in Art, Music and Science*, 47-56.
- Puckette, M. (2007). *The Theory and Technique of Electronic Music*. World Scientific.

### Hardware Documentation

- Sequentix Cirklon Manual: [https://sequentix.com/cirklon](https://sequentix.com/cirklon)
- Plinkysynth Documentation: [https://www.plinkysynth.com](https://www.plinkysynth.com)

---

## 9. License

MIT License (2025)

Copyright (c) 2025 George Redpath

---

## 10. Author

**George Redpath** (Ziforge)
GitHub: [@Ziforge](https://github.com/Ziforge)
Focus: Hardware synthesis, MIDI sequencing, generative music systems

---

## 11. Acknowledgments

- **Sequentix** — Cirklon hardware sequencer design
- **Plinkysynth** — Touch-plate synthesizer development
- **MIDI Manufacturers Association** — MIDI protocol standardization

---

## 12. Citation

```bibtex
@misc{redpath2025cirklon,
  author = {Redpath, George},
  title = {Sequentix Cirklon Instrument Definitions for Plinky Synthesizer},
  year = {2025},
  publisher = {GitHub},
  url = {https://github.com/Ziforge/CirklonSynthDefs}
}
```

---

Built for hardware synthesis integration and MIDI sequencing workflows.
