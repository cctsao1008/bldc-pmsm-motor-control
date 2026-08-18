# Software Architecture

The software architecture separates hardware-dependent timing and signal acquisition from sensing, motor state, control algorithms, modulation, and supervisory behavior.

## Separate hardware from control algorithms

The intended control flow is:

```text
Motor / Inverter
      ↓
Sensing / Scaling
      ↓
State / Position Information
      ↓
Control Algorithm
      ↓
Modulation / Commutation
      ↓
PWM / Gate Driver
      ↓
Motor / Inverter
```

Conceptually, the implementation is divided into:

```text
hardware/       pwm, adc, gpio, timing, protection
sensing/        current, bus_voltage, hall, bemf
motor/          parameters, electrical_angle, scaling
control/
    six_step/   startup, bemf, commutation, speed_control
    foc/        clarke, park, current_control, svpwm
supervisor/     startup, mode_selection, faults, shutdown
```

The MCU is an implementation target, not the architectural identity of the project.

## Hardware layer

The hardware layer owns MCU-specific behavior such as:

- complementary PWM generation;
- effective dead time;
- timer synchronization;
- ADC triggering;
- DMA where applicable;
- GPIO and Hall capture;
- hardware fault inputs;
- emergency PWM shutdown.

Higher layers should not depend directly on MCU timer or ADC register layouts.

## Sensing layer

The sensing layer converts raw hardware measurements into engineering quantities:

- `ISA`, `ISB`, `ISC` → phase currents;
- `VAD` → DC-bus voltage;
- `HALLA`, `HALLB`, `HALLC` → rotor sector / position information;
- `EMFA`, `EMFB`, `EMFC` or `EOA`, `EOB`, `EOC` → back-EMF information.

Calibration, offsets, gains, filtering, blanking, and measurement validity belong here rather than inside the control law.

## Motor layer

The motor layer contains physical motor parameters and state transformations shared by controllers, such as:

- pole-pair count;
- phase resistance and inductance;
- electrical-angle conversion;
- mechanical/electrical speed conversion;
- current and voltage scaling.

## Control layer

Two primary control paths are maintained independently.

### Sensorless six-step

```text
Startup / Alignment
      ↓
Open-Loop Commutation
      ↓
BEMF Qualification
      ↓
Zero-Cross Detection
      ↓
Commutation Timing
      ↓
Six-Step Switching
```

### Field-Oriented Control

```text
Ia / Ib / Ic
     ↓
Clarke
     ↓
Park
     ↓
Id / Iq PI
     ↓
Inverse Park
     ↓
SVPWM
     ↓
PWM
```

The two paths may share hardware and sensing services, but should not be coupled through implementation-specific assumptions.

## Supervisor layer

The supervisor owns system-level state and safety behavior:

- initialization;
- precharge or startup sequencing where required;
- operating-mode selection;
- open-loop to closed-loop handover;
- fault latching;
- controlled shutdown;
- restart policy.

This separation keeps algorithm development from bypassing system-level protection and transition logic.
