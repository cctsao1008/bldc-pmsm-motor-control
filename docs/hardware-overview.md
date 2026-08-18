# Hardware Overview

The initial platform is a three-phase BLDC/PMSM inverter board intended for external-MCU control.

## Power Stage

The power stage is a three-phase, six-switch inverter:

```text
                VM
                 │
       ┌─────────┼─────────┐
       │         │         │
      Q1        Q3        Q5
       │         │         │
      MA        MB        MC
       │         │         │
      Q2        Q4        Q6
       │         │         │
     R42       R43       R44
     1 mΩ      1 mΩ      1 mΩ
       │         │         │
      GND       GND       GND
```

Nominal board data:

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

These values describe the initial hardware reference and do not constrain later controller implementations.

## PWM / Gate Drive

The board exposes independent high-side and low-side control for each phase:

```text
Phase A: INHA / INLA
Phase B: INHB / INLB
Phase C: INHC / INLC
```

The external MCU is responsible for complementary PWM generation, dead time, timer synchronization, ADC trigger timing, safe disabled states, and fault shutdown behavior.

The hardware should be characterized from MCU output through the full switching chain:

```text
MCU PWM
   ↓
Gate-Driver Input
   ↓
MOSFET Gate
   ↓
Phase Switching Node
```

PWM polarity and effective dead time must be verified on hardware before normal motor operation.

## Motor Outputs

The three motor phases are exposed as:

```text
MA
MB
MC
```

Phase order and rotation direction must be verified during bring-up rather than assumed from labeling alone.

## External MCU Interface

The board provides the signals required for several motor-control strategies:

- six PWM control inputs;
- three phase-current outputs (`ISA`, `ISB`, `ISC`);
- Hall inputs (`HALLA`, `HALLB`, `HALLC`);
- analog back-EMF signals (`EMFA`, `EMFB`, `EMFC`);
- comparator back-EMF outputs (`EOA`, `EOB`, `EOC`);
- DC-bus voltage feedback (`VAD`);
- logic supply and ground connections.

The MCU is therefore an implementation choice rather than part of the architectural identity of the project.
