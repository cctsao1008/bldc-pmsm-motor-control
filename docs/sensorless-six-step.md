# Sensorless Six-Step Control

Sensorless six-step commutation is one of the two primary control paths in this project.

## Operating Principle

During steady-state six-step operation, two phases are driven and the third phase is left floating. The floating phase is observed for back-EMF zero-cross information.

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

The board provides both analog phase-related signals (`EMFA`, `EMFB`, `EMFC`) and LM339 comparator outputs (`EOA`, `EOB`, `EOC`) for this purpose.

## Startup

Back-EMF approaches zero as motor speed approaches zero, so zero-cross detection cannot provide rotor position at standstill.

Startup therefore requires a separate sequence:

```text
Rotor Alignment
      ↓
Open-Loop Commutation
      ↓
Acceleration
      ↓
BEMF Qualification
      ↓
Sensorless Handover
      ↓
Closed-Loop Commutation
```

The startup strategy must be characterized rather than treated as a fixed delay sequence.

Important parameters include:

- alignment duty and duration;
- initial commutation period;
- acceleration ramp;
- current during startup;
- minimum valid BEMF amplitude;
- minimum reliable takeover speed.

## Zero-Cross Detection

A useful zero-cross detector must distinguish real back-EMF events from switching transients and comparator noise.

Development items include:

- commutation blanking time;
- BEMF polarity and phase mapping;
- edge qualification;
- noise rejection;
- expected sector sequence;
- zero-cross-to-commutation delay;
- timeout and loss-of-sync detection.

## Six Electrical Sectors

The controller cycles through six commutation states:

```text
1 → 2 → 3 → 4 → 5 → 6 → 1
```

At each state:

- one phase is driven high;
- one phase is driven low;
- one phase is floating and available for BEMF observation.

The final switching table should be documented only after motor phase order, signal polarity, and rotation direction have been verified on hardware.

## Validation Targets

Key sensorless six-step measurements include:

- startup success rate;
- startup time;
- minimum reliable sensorless speed;
- BEMF waveform quality;
- zero-cross timing repeatability;
- commutation timing error;
- speed ripple;
- response to load changes;
- loss-of-sync behavior;
- restart behavior after a fault or stall.
