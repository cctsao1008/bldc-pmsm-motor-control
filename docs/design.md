# Design

This project treats the motor, inverter, gate drive, sensing paths, timing, and mechanical load as one physical control system.

## Physical plant first

Motor-control software is developed from the actual electrical and mechanical plant:

```text
DC Supply
    ↓
DC Bus
    ↓
Three-Phase Inverter
    ↓
BLDC / PMSM Motor
    ↓
Mechanical Load
```

The effective plant also includes:

```text
PWM Timing
    +
Gate Driver
    +
MOSFET Switching
    +
Phase-Current Measurement
    +
Back-EMF / Hall Feedback
    +
Motor Electrical Dynamics
    +
Motor Mechanical Dynamics
```

The inverter and motor are therefore not treated as ideal mathematical blocks detached from their hardware implementation.

Development follows an iterative measurement-driven process:

```text
Physical System
      ↓
Measurement
      ↓
Characterization / Modeling
      ↓
Control Design
      ↓
Firmware Implementation
      ↓
Hardware Test
      ↓
Model / Parameter Update
      ↺
```

A model is useful only to the extent that it predicts the real hardware closely enough to support control design and explain measured behavior.

## Measurement before claims

A motor spinning is not by itself considered successful control.

Behavior should be:

- measurable;
- explainable;
- reproducible;
- comparable against a defined baseline.

For sensorless six-step control, this includes actual phase voltage, back-EMF zero crossings, commutation timing, startup behavior, and usable speed range.

For FOC, this includes phase-current acquisition, ADC timing, current reconstruction, `Id` / `Iq` behavior, modulation, and closed-loop response.

## Development principle

The project keeps the physical plant and experimental conditions stable enough that different control approaches can be evaluated meaningfully on the same hardware.

Primary control paths are:

- sensorless six-step commutation;
- Hall-assisted or sensored commutation where useful for characterization;
- field-oriented control (FOC);
- later sensorless FOC and observer-based approaches.

The objective is not to declare one algorithm universally superior, but to expose engineering trade-offs under repeatable conditions.
