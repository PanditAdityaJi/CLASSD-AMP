# Class-D Audio Amplifier — NE555 PWM

A low-power Class-D audio amplifier built around the NE555 timer as a PWM generator. The design covers the full signal path from microphone/line input to an 8 Ω speaker, with LTspice simulation and a KiCad PCB layout included.

---

## How It Works

```
Audio In ──► opamp ──► NE555 ──► BJT Driver ──► PMOS/NMOS ──► LC Filter ──► Speaker
            Comparator   PWM      (Q1/Q2)        Half-Bridge    33µH/470nF
```

The audio signal and a triangle carrier are compared by an op-amp running open-loop. The comparator output triggers the NE555, which produces a PWM waveform whose duty cycle tracks the audio amplitude. A complementary BJT pair buffers the PWM to fast-charge the MOSFET gates of an (PMOS) /  (NMOS) half-bridge. An LC low-pass filter strips the switching harmonics, leaving a clean audio signal for the speaker.


## NE555 Pin Connections

| Pin | Name | Connection |
|-----|------|------------|
| 1 | GND  | Ground plane |
| 2 | TRIG | Triangle carrier via 1 nF AC-coupling cap (C2) |
| 3 | OUT  | PWM → gate driver through 100 Ω series resistor (R6) |
| 4 | RST  | Tied directly to VCC — **never** to OUT |
| 5 | CV   | 10 nF cap (C3) to GND — decouples internal divider |
| 6 | THRS | Timing capacitor + R3/R4 network |
| 7 | DIS  | Between R3 (1 kΩ) and R4 (3.3 kΩ) |
| 8 | VCC  | 12 V supply |

---

## Key Design Decisions

**Why NE555 instead of a microcontroller?** Frequency is set entirely by passive components — no firmware, no clock dependency.

**Why a BJT driver stage?** Power MOSFETs have several nF of gate capacitance. Driving them straight from the 555 output produces slow edges and shoot-through overlap. The emitter follower charges and discharges the gates in tens of nanoseconds.

**Why PMOS + NMOS instead of dual N-channel with a bootstrap?** Simpler gate drive — no floating supply needed for the high side. The pair has low enough R_DS(on) (0.5–1.5 Ω) that conduction loss is small against the 8 Ω load.

**Why 100 Ω on the gate (R6)?** Without it the gate trace resonates with the MOSFET input capacitance at hundreds of MHz. 100 Ω damps the ring without slowing the edge enough to increase switching loss.

**Why 33 µH / 0.47 µF for the LC filter?** The cut-off lands at roughly 300 kHz, giving >40 dB/decade roll-off. Film caps were specified over ceramics because ceramics of this value suffer severe DC-bias derating that would shift the pole under signal swing.

**Why matched R5 = R8 = 1 kΩ in the triangle generator?** Equal resistors make the rising and falling slopes identical, producing a symmetric carrier. An asymmetric carrier introduces even-order harmonics and a DC offset at the speaker.

---

## Bill of Materials

| Ref | Part | Value | Notes |
|-----|------|-------|-------|
| U1 | NE555D | — | SOIC-8 |
| U2 | LMV321 | — | SOT-23-5, open-loop comparator |
| Q1 | 2N3904 | — | NPN gate driver |
| Q2 | 2N3906 | — | PNP gate driver |
| Q3 | IRF9540 | — | PMOS high-side switch |
| Q4 | IRF540  | — | NMOS low-side switch |
| L1 | Inductor | 33 µH | Ferrite core, ≥ 3 A rated |
| R1 | Resistor | 100 kΩ | Input bias divider |
| R2 | Resistor | 100 kΩ | Input bias divider |
| R3 | Resistor | 1 kΩ   | 555 timing (charge) |
| R4 | Resistor | 3.3 kΩ | 555 timing (discharge) |
| R5 | Resistor | 1 kΩ   | Triangle generator |
| R6 | Resistor | 100 Ω  | Gate series damping |
| R7 | Resistor | 8 Ω    | Speaker load |
| R8 | Resistor | 1 kΩ   | Triangle generator |
| C1 | Capacitor | 10 nF  | Audio input AC-coupling |
| C2 | Capacitor | 1 nF   | TRIG AC-coupling |
| C3 | Capacitor | 10 nF  | CV pin bypass |
| C4 | Capacitor | 10 nF  | Triangle timing |
| C5 | Capacitor | 0.47 µF | LC filter (film) |
| C6 | Capacitor | 0.47 µF | Output filter / DC-block (film) |
| J2 | Connector | 2-pin | Power input (24 V) |
| J3 | Connector | 2-pin | Audio input |
| J4 | Connector | 2-pin | Speaker output |
| LS1 | Speaker | 8 Ω | — |

---

## Simulation

Simulated end-to-end in LTspice with a 1 kHz / 0.5 V sine test tone.

**Verified nodes:**

- **V(triangle)** — symmetric carrier, 3.4–3.6 V swing at the carrier frequency.
- **V(gate)** — clean PWM from the 555 output stage.
- **V(switching)** — half-bridge output, full rail-to-rail (0 V ↔ 24 V).
- **V(audio_out)** — filtered output; residual ripple is only a few millivolts.

**FFT results:**

- Audio_out shows the 1 kHz tone near −55 dB with harmonics below −140 dB across the audio band.
- Switching node shows the expected carrier and odd harmonics near 100 kHz.

---
