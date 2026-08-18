# Field-Oriented Control

Field-Oriented Control (FOC) is the second primary control path in this project.

## Control Chain

The intended current-control flow is:

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

## Phase-Current Acquisition

The board uses three low-side shunts and INA181A1 amplifiers. Because valid measurement windows depend on switching state, current acquisition must be synchronized to PWM timing.

FOC development therefore depends on the current-sensing path being validated first:

- zero-current offset;
- gain and polarity;
- ADC scaling;
- PWM trigger point;
- switching transient rejection;
- current reconstruction validity.

## Current Control

For a typical surface-mounted PMSM below field-weakening speed, an initial operating target is:

```text
Id* ≈ 0
Iq* → torque command
```

The inner loops regulate:

```text
Id → Id*
Iq → Iq*
```

using PI controllers in the rotating `dq` reference frame.

Controller gains should ultimately be related to motor electrical parameters, loop update rate, delay, and measured plant response rather than tuned only by trial and error.

## Rotor Electrical Angle

FOC requires rotor electrical angle:

```text
θe = pole_pairs × θm
```

Possible angle sources include:

- Hall sensors;
- external encoder;
- observer-based estimation;
- later sensorless FOC methods.

Hall or encoder feedback can be used initially to establish a known-good FOC baseline before introducing sensorless angle estimation.

## SVPWM

Space Vector PWM maps the commanded stationary-frame voltage vector into three-phase switching times.

The implementation must account for:

- DC-bus voltage;
- timer resolution;
- modulation limits;
- dead time;
- minimum pulse width;
- ADC sampling-window requirements;
- saturation behavior.

## Validation Targets

FOC validation includes:

- current reconstruction error;
- `Id` tracking and steady-state offset;
- `Iq` tracking and torque response;
- current-loop bandwidth;
- phase-current ripple;
- speed-loop response;
- modulation saturation;
- low-speed behavior;
- thermal rise;
- computational cost.

Sensorless FOC is treated as a later extension and should not be conflated with the initial sensorless six-step implementation.
