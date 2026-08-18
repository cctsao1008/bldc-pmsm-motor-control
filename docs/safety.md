# Safety

This project controls a three-phase inverter capable of significant current and stored energy. Incorrect PWM polarity, insufficient dead time, uncontrolled startup, invalid current feedback, or software faults can damage the inverter, motor, power supply, measurement equipment, or connected systems.

## Primary Risks

Particular attention must be given to:

- DC input polarity;
- shoot-through prevention;
- effective dead-time verification;
- current limiting;
- safe startup and shutdown;
- regenerative DC-bus voltage rise;
- motor overcurrent;
- MOSFET and shunt temperature;
- hardware fault handling;
- oscilloscope grounding.

## Initial Power-Up

Use conservative conditions during bring-up:

```text
Reduced DC-bus voltage
Current-limited bench supply
Motor disconnected for initial rail checks
PWM disabled by default
Unloaded motor where possible
Oscilloscope verification before closed-loop operation
```

The board is specified for 12–36 V input, but initial bring-up should not begin at the maximum input voltage.

## PWM / Gate-Drive Safety

Before enabling normal motor operation, verify:

- high-side and low-side polarity;
- complementary relationship;
- effective dead time at the MOSFET gates;
- disabled-state behavior;
- emergency shutdown path;
- absence of unintended simultaneous conduction.

Do not rely only on MCU timer configuration values; verify the resulting gate waveforms directly.

## Regenerative Behavior

A spinning motor can return energy to the DC bus during deceleration or braking. The resulting bus voltage can exceed the supply setting if the source cannot absorb returned energy.

Monitor `VAD` and the actual DC bus during braking and rapid deceleration tests.

## Measurement Safety

Standard earth-referenced oscilloscope probe grounds must not be connected indiscriminately to switching nodes or floating nodes.

Use differential or properly isolated measurement methods where required, and verify the grounding relationship between:

- bench supply;
- oscilloscope;
- MCU debugger;
- USB-connected equipment;
- motor-control board.

## Fault Policy

Software protection should fail toward a safe PWM-disabled state. Fault handling should include, where applicable:

- overcurrent;
- overvoltage;
- undervoltage;
- invalid current measurement;
- loss of commutation or rotor position;
- excessive temperature;
- startup timeout;
- watchdog or control-loop timing failure.

Protection behavior should be verified experimentally rather than assumed from code inspection alone.
