# BLDC / PMSM Motor Control

Embedded motor-control development project for BLDC and PMSM drives, covering sensorless six-step commutation, three-phase current sensing, Hall and back-EMF feedback, and field-oriented control (FOC).

## Why This Project

The initial hardware platform is an existing three-phase inverter board with gate drivers, three-shunt low-side current sensing, Hall inputs, back-EMF sensing, and DC-bus voltage feedback.

The objective is not simply to make a motor spin or reproduce vendor firmware. The board is used as a real physical motor-control platform for understanding how switching hardware, sensing, timing, estimation, commutation, and closed-loop control interact.

Two major control paths are developed on the same hardware:

- **Sensorless six-step commutation** using back-EMF zero-cross detection;
- **Field-Oriented Control (FOC)** using synchronized phase-current measurement and vector control.

The project focuses on:

- reconstructing and documenting the actual power stage and control interfaces;
- establishing reproducible hardware measurements;
- developing independent motor-control firmware;
- validating PWM, gate-drive, sensing, and commutation timing on real hardware;
- implementing sensorless six-step control;
- implementing current-controlled FOC and SVPWM;
- comparing control strategies under consistent electrical and mechanical conditions.

The long-term objective is to build a reusable BLDC/PMSM motor-control implementation in which algorithms can be evaluated against real inverter and motor behavior rather than simulation alone.

## Design

The detailed design principles are documented separately to keep this README focused on the project overview.

- [Design principles](docs/design.md) — physical-plant-first development, measurement-driven validation, and experimental discipline.
- [Software architecture](docs/software-architecture.md) — separation of hardware-dependent PWM/ADC/timing from sensing, motor state, control algorithms, and supervisory logic.

## System

The initial hardware is a three-phase BLDC/PMSM inverter with six PWM-controlled MOSFET switches and multiple feedback paths.

The main signal paths are:

```text
                       ┌─────────────────┐
                       │       MCU       │
                       └────────┬────────┘
                                │
                         6-PWM Gate Drive
                                │
                                ▼
                    ┌───────────────────────┐
                    │ Three-Phase Inverter  │
                    └───────────┬───────────┘
                                │
                          MA / MB / MC
                                │
                                ▼
                         BLDC / PMSM
                                │
             ┌──────────────────┼──────────────────┐
             │                  │                  │
             ▼                  ▼                  ▼
       Phase Current          Hall              BEMF
        INA181 ×3           Feedback          Feedback
             │                  │                  │
             └──────────────────┼──────────────────┘
                                ▼
                               MCU
```

The same power stage supports both primary control paths:

```text
                     Motor-Control Platform
                              │
              ┌───────────────┴───────────────┐
              │                               │
              ▼                               ▼
     Sensorless Six-Step                     FOC
              │                               │
         BEMF ZC                    Phase Current + Angle
              │                               │
       Commutation Timing            Clarke / Park / PI
              │                               │
          Six States                         SVPWM
              └───────────────┬───────────────┘
                              ▼
                    Three-Phase Inverter
```

## Hardware Reference

The initial board has the following nominal configuration:

| Parameter | Value |
| --- | --- |
| Motor type | BLDC / PMSM |
| Power-stage topology | Three-phase, six-switch inverter |
| Input voltage | 12–36 VDC |
| Continuous output current | 10 A |
| Maximum output current | 20 A |
| Nominal maximum power | 500 W |
| Main MOSFET | HYG065N07 ×6 |
| PWM interface | 6-PWM |
| Phase current sensing | Three-shunt low-side |
| Current-sense amplifier | INA181A1 ×3 |
| Current shunt | 1 mΩ ×3 |
| Current-sense reference | 1.65 V |
| Hall feedback | HALLA / HALLB / HALLC |
| Back-EMF sensing | EMFA / EMFB / EMFC |
| BEMF comparator | LM339 |
| Comparator outputs | EOA / EOB / EOC |
| DC-bus voltage feedback | VAD |
| MCU | External |

These parameters describe the initial hardware reference and do not constrain future controller implementations.

## Sensing Architecture

### Phase-current sensing

The board uses three independent low-side shunts:

```text
Phase A → R42 = 1 mΩ → GND
Phase B → R43 = 1 mΩ → GND
Phase C → R44 = 1 mΩ → GND
```

Each shunt is measured by an INA181A1 current-sense amplifier:

```text
R42 → INA181A1 → ISA
R43 → INA181A1 → ISB
R44 → INA181A1 → ISC
```

The amplifiers use approximately `REF1V65 = 1.65 V` as the zero-current reference.

Because the shunts are located below the low-side MOSFETs, valid phase-current measurement depends on switching state. ADC acquisition therefore requires PWM-synchronized sampling rather than arbitrary asynchronous reads.

For a three-wire motor without neutral connection:

```text
Ia + Ib + Ic ≈ 0
```

provides a useful measurement-consistency check.

### Back-EMF sensing

The three motor phases provide back-EMF related signals:

```text
MA → EMFA
MB → EMFB
MC → EMFC
```

A comparator stage based on the LM339 provides corresponding digital outputs:

```text
EMFA → EOA
EMFB → EOB
EMFC → EOC
```

These signals provide the basis for sensorless six-step zero-cross detection and commutation timing.

### Hall sensing

The board exposes:

```text
HALLA
HALLB
HALLC
```

for sensored commutation, rotor-sector validation, startup characterization, and comparison with BEMF-based operation.

### DC-bus voltage sensing

The DC bus is monitored through `VAD` for undervoltage/overvoltage detection, regenerative behavior, modulation normalization, electrical power estimation, and protection logic.

## Control Paths

### Sensorless Six-Step

The steady-state operating sequence is conceptually:

```text
Commutation
     ↓
Floating Phase
     ↓
Back-EMF Measurement
     ↓
Zero-Cross Detection
     ↓
Electrical Timing Estimate
     ↓
Next Commutation Event
     ↺
```

Because back-EMF approaches zero at standstill, startup requires a separate open-loop sequence:

```text
Rotor Alignment
      ↓
Open-Loop Commutation
      ↓
Acceleration
      ↓
Detect Valid BEMF
      ↓
Closed-Loop Sensorless Commutation
```

Important development targets include minimum reliable takeover speed, startup current, acceleration profile, false zero-cross rejection, blanking time, commutation delay, and behavior under load.

### Field-Oriented Control

The FOC control chain is:

```text
Ia / Ib / Ic
     ↓
Clarke Transform
     ↓
Iα / Iβ
     ↓
Park Transform
     ↓
Id / Iq
     ↓
Current PI Controllers
     ↓
Vd / Vq
     ↓
Inverse Park Transform
     ↓
Vα / Vβ
     ↓
SVPWM
     ↓
Three-Phase Inverter
```

For a typical surface-mounted PMSM operating below field-weakening speed:

```text
Id* ≈ 0
Iq* → torque command
```

FOC also requires rotor electrical angle:

```text
θe = pole_pairs × θm
```

Possible angle sources include Hall sensors, an external encoder, and later observer-based sensorless estimation.

Sensorless FOC is treated as a later extension and is distinct from the initial sensorless six-step implementation.

## Reference Baseline

Existing vendor material provides useful known-working references for the hardware, including examples based on:

- STM32F405 and VESC-derived firmware;
- ESP32 and SimpleFOC;
- AT32 sensorless six-step control;
- STM32F103 six-step and sensored motor-control examples.

These implementations are treated as reference material rather than the architectural basis of this project.

Reference firmware can be used to answer questions such as:

- Does the power stage operate correctly?
- Is phase wiring correct?
- Are the sensing paths functional?
- Can the selected motor run on the board?
- What waveforms represent known-working operation?

Vendor firmware, documentation, and third-party source code are not redistributed unless their respective licenses explicitly permit redistribution.

## Bring-Up Sequence

### Stage 0 — Power rails

Motor disconnected, PWM disabled:

```text
VM
12 V rail
5 V rail
3.3 V rail
REF1V65
ISA / ISB / ISC
VAD
```

Initial power-up should use a current-limited bench supply at a reduced DC-bus voltage.

### Stage 1 — PWM and gate drive

Verify PWM polarity, high-side/low-side relationship, effective dead time, disabled state, gate amplitude, and switching-node waveform.

### Stage 2 — Open-loop inverter operation

Verify phase order, six-step sequence, motor direction, and basic PWM control.

### Stage 3 — Current sensing

Verify zero-current offsets, gain, ADC timing, phase-current polarity, and `Ia + Ib + Ic` consistency.

### Stage 4 — Sensorless six-step

Develop startup, BEMF acquisition, zero-cross detection, commutation timing, and speed control.

### Stage 5 — FOC

Develop synchronized current sampling, Clarke/Park transforms, `Id` / `Iq` current control, SVPWM, and rotor-angle input.

### Stage 6 — Advanced estimation and control

Possible extensions include sensorless FOC, observers, PLL-based angle estimation, motor parameter identification, field weakening, and MTPA where applicable.

## Validation

### Electrical validation

- PWM frequency and effective dead time;
- MOSFET gate and phase-node waveforms;
- DC-bus ripple;
- current-sense offset and gain;
- phase-current waveform and switching noise;
- back-EMF waveform;
- Hall timing.

### Sensorless six-step validation

- startup success rate;
- startup time;
- minimum sensorless speed;
- BEMF zero-cross quality;
- commutation timing error;
- speed ripple;
- load-disturbance response;
- loss-of-sync behavior.

### FOC validation

- ADC sampling position;
- current reconstruction error;
- `Id` and `Iq` regulation;
- current-loop bandwidth;
- phase-current ripple;
- speed-loop response;
- modulation saturation;
- low-speed behavior;
- torque response.

### Thermal and protection validation

- MOSFET, shunt, and gate-driver temperature rise;
- continuous-current capability;
- overcurrent response;
- DC-bus overvoltage;
- regenerative behavior.

## Controller Benchmark

Different control strategies should be compared under common operating conditions.

| Metric | Description |
| --- | --- |
| Startup success | Successful starts / total attempts |
| Startup time | Time to stable closed-loop operation |
| Minimum operating speed | Lowest stable closed-loop speed |
| Speed ripple | Steady-state speed variation |
| Speed response | Rise and settling behavior |
| Phase-current ripple | Electrical current quality |
| Peak phase current | Semiconductor and motor stress |
| Torque response | Response to torque command / load |
| Commutation stability | Six-step timing robustness |
| `Id` regulation | FOC flux-axis tracking |
| `Iq` regulation | FOC torque-axis tracking |
| DC-bus ripple | Supply / regenerative behavior |
| Efficiency | Electrical input vs mechanical/electrical output |
| Thermal rise | Power-stage thermal behavior |
| Robustness | Sensitivity to motor / load / voltage variation |
| Computational cost | CPU time and memory requirement |

The objective is to expose engineering trade-offs rather than declare one control strategy universally superior.

## Safety

This project controls a three-phase inverter capable of significant current and stored energy. Incorrect PWM polarity, insufficient dead time, uncontrolled startup, invalid current feedback, or software faults can destroy the inverter, motor, power supply, or connected equipment.

Particular attention must be given to:

- DC input polarity;
- shoot-through prevention;
- effective dead-time verification;
- current limiting;
- safe startup and shutdown;
- regenerative bus-voltage rise;
- motor overcurrent;
- MOSFET and shunt temperature;
- hardware fault handling;
- oscilloscope grounding.

Initial testing should use a reduced DC-bus voltage, current-limited bench supply, unloaded motor where possible, PWM disabled by default, and oscilloscope verification before closed-loop operation.

## Status

Early-stage project.

Current focus: **hardware documentation, power-stage characterization, sensing validation, and establishment of a known-good motor-control baseline before independent sensorless six-step and FOC development.**

## License

To be determined.
