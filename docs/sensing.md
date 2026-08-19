# Sensing Architecture

The initial board exposes four sensing paths relevant to motor control: three-phase current, back-EMF, Hall feedback, and DC-bus voltage.

The board is treated as a vendor-validated, known-good hardware baseline. The sensing work in this project therefore focuses on controller-side acquisition, scaling, timing, calibration, and interpretation of these signals.

## Phase-Current Sensing

The board uses three independent low-side shunts:

```text
Phase A → R42 = 5 mΩ → GND
Phase B → R43 = 5 mΩ → GND
Phase C → R44 = 5 mΩ → GND
```

Each shunt is measured by an INA181A1 current-sense amplifier:

```text
R42 → INA181A1 → ISA
R43 → INA181A1 → ISB
R44 → INA181A1 → ISC
```

The amplifiers use approximately:

```text
REF1V65 = 1.65 V
```

as the zero-current reference.

Actual offset, gain, ADC scaling, amplifier error, shunt tolerance, switching interference, and temperature drift must be measured and calibrated at the controller level.

### PWM-Synchronized Sampling

Because the current shunts are below the low-side MOSFETs, phase-current measurement validity depends on inverter switching state.

ADC acquisition therefore requires PWM-synchronized sampling:

```text
PWM Timer
    │
    ├── MOSFET switching
    │
    └── ADC trigger
            ↓
      Valid sampling window
            ↓
       ISA / ISB / ISC
            ↓
      Current reconstruction
```

ADC trigger position, blanking time, switching transients, amplifier settling, and minimum measurable PWM windows are part of the effective sensing system.

For a three-wire motor without a neutral connection:

```text
Ia + Ib + Ic ≈ 0
```

provides a useful measurement-consistency check. A non-zero residual can indicate offset mismatch, gain mismatch, invalid sampling, or switching interference.

## Back-EMF Sensing

The three phase nodes provide analog back-EMF-related signals:

```text
MA → EMFA
MB → EMFB
MC → EMFC
```

The board also provides a common virtual-neutral/reference signal, `VN`, used by the back-EMF comparator stage.

Conceptually:

```text
MA / MB / MC
     ↓
Phase-derived BEMF signals
     ↓
EMFA / EMFB / EMFC
     │
     ├── analog observation
     │
VN ──┴── LM339 comparison
             ↓
         EOA / EOB / EOC
```

These signals form the hardware basis for sensorless six-step zero-cross detection and commutation timing.

The useful quantity is not simply whether a comparator toggles, but whether the detected zero-cross event can be related consistently to the correct electrical commutation instant under changing speed, load, PWM duty, and switching noise.

## Hall Sensing

The board exposes:

```text
HALLA
HALLB
HALLC
```

These provide discrete rotor-position feedback to the controller. Their exact phase/sector mapping, polarity, and timing relationship should be established during controller integration before being used by sensored commutation or rotor-angle logic.

## DC-Bus Voltage Sensing

The DC bus is monitored through `VAD`:

```text
VM
 ↓
Voltage Divider
 ↓
VAD
 ↓
ADC
```

Bus-voltage measurement may be used for:

- undervoltage detection;
- overvoltage detection;
- regenerative bus-rise observation;
- modulation normalization;
- electrical power estimation;
- protection and supervisory logic.
