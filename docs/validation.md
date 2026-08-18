# Validation and Benchmarking

Control strategies are evaluated under common electrical and mechanical conditions. The objective is to expose engineering trade-offs rather than declare one method universally superior.

## Electrical Validation

Verify and record:

- PWM frequency;
- effective dead time;
- MOSFET gate waveforms;
- phase-node waveforms;
- DC-bus ripple;
- current-sense offset and gain;
- phase-current waveforms;
- switching noise;
- back-EMF waveforms;
- Hall timing.

## Sensorless Six-Step Validation

Measure:

- startup success rate;
- startup time;
- minimum sensorless speed;
- BEMF zero-cross quality;
- zero-cross timing repeatability;
- commutation timing error;
- speed ripple;
- response to load disturbance;
- loss-of-sync behavior;
- recovery and restart behavior.

## FOC Validation

Measure:

- ADC sampling position;
- current reconstruction error;
- `Id` regulation;
- `Iq` regulation;
- current-loop bandwidth;
- phase-current ripple;
- speed-loop response;
- modulation saturation;
- low-speed behavior;
- torque response.

## Thermal and Protection Validation

Measure or verify:

- MOSFET temperature rise;
- shunt temperature rise;
- gate-driver temperature;
- continuous-current capability;
- overcurrent response;
- DC-bus overvoltage behavior;
- regenerative behavior;
- controlled shutdown and fault latching.

## Common Benchmark Metrics

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
| Efficiency | Electrical input versus delivered output |
| Thermal rise | Power-stage thermal behavior |
| Robustness | Sensitivity to motor, load, and voltage variation |
| Computational cost | CPU time and memory requirement |

## Measurement Discipline

Each result should record enough context to be reproduced later, including where applicable:

- hardware revision;
- firmware commit;
- MCU and clock configuration;
- motor identity and parameters;
- supply voltage and current limit;
- PWM frequency;
- control-loop rate;
- load condition;
- measurement instrument and probe method;
- test date and test procedure.
