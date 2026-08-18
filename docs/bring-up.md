# Hardware Bring-Up

Bring-up proceeds in increasing order of stored energy, switching risk, and control complexity.

## Stage 0 — Power Rails

Motor disconnected and PWM disabled.

Verify:

```text
VM
12 V rail
5 V rail
3.3 V rail
REF1V65
ISA / ISB / ISC
VAD
```

Initial power-up should use a reduced DC-bus voltage and a current-limited bench supply.

## Stage 1 — PWM and Gate Drive

Verify:

- PWM polarity;
- high-side / low-side relationship;
- effective dead time;
- disabled state;
- gate amplitude;
- phase switching-node waveform;
- emergency PWM shutdown behavior.

Do not proceed until complementary switching has been verified directly on hardware.

## Stage 2 — Open-Loop Inverter Operation

Verify:

- phase order;
- six-step commutation sequence;
- motor rotation direction;
- basic PWM duty control;
- no unexpected shoot-through or DC-bus current spikes.

## Stage 3 — Current Sensing

Verify:

- zero-current offsets;
- current-sense gain;
- ADC scaling;
- ADC trigger timing;
- phase-current polarity;
- switching transient rejection;
- `Ia + Ib + Ic` consistency.

## Stage 4 — Sensorless Six-Step

Develop and validate:

- rotor alignment;
- open-loop startup;
- acceleration profile;
- BEMF acquisition;
- zero-cross qualification;
- blanking time;
- commutation timing;
- open-loop to sensorless handover;
- speed control.

## Stage 5 — Field-Oriented Control

Develop and validate:

- synchronized current acquisition;
- Clarke and Park transforms;
- electrical-angle interface;
- `Id` / `Iq` current control;
- inverse Park transform;
- SVPWM;
- current-loop and speed-loop behavior.

## Stage 6 — Advanced Estimation and Control

Possible later work includes:

- sensorless FOC;
- observer-based rotor-angle estimation;
- PLL-based estimation;
- motor parameter identification;
- field weakening;
- MTPA where applicable.

## Bring-Up Rule

A later stage should not be used to hide an unresolved problem in an earlier stage. For example, control-loop tuning should not be used to compensate for incorrect current polarity, invalid ADC timing, or unverified gate-drive timing.
