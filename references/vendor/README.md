# Vendor Reference Material

This directory contains vendor-provided reference material for the initial BLDC/PMSM motor-control hardware platform.

The hardware is treated as a vendor-validated, known-good baseline. These files are retained as external reference material; project-maintained engineering interpretation belongs under `docs/`.

## Hardware References

| File | Date | Source | Role |
| --- | --- | --- | --- |
| `hardware/driver-board-pinout.png` | — | Vendor-provided hardware documentation | Physical connector and signal reference for the driver board |
| `hardware/low-side-current-sense-driver-board-overview.png` | — | Vendor-provided hardware documentation | Board-level overview of the low-side current-sensing inverter platform |
| `hardware/low-side-current-sense-driver-board-schematic-2026-04-22.pdf` | 2026-04-22 | Vendor-provided schematic | Primary hardware reference for the three-phase inverter, low-side current sensing, back-EMF sensing, and board power rails |
| `hardware/vesc75-controller-board-schematic-2025-12-29.pdf` | 2025-12-29 | Vendor-provided VESC75-compatible controller documentation | Controller-board reference for STM32F405 signal mapping, PWM, ADC, Hall, back-EMF, USB, UART, and debug interfaces |

## VESC Reference Ecosystem

The VESC-related vendor material in this repository should be treated as a vendor implementation/reference, not automatically as official VESC hardware.

The official VESC project and ecosystem reference is:

- VESC Project — https://vesc-project.com/

Accordingly, provenance should be interpreted as:

```text
VESC Project
    ↓
official ecosystem / architecture reference

Vendor-provided VESC75-compatible material
    ↓
board-specific implementation reference
```

## Repository Role

The intended information flow is:

```text
Vendor reference material
        ↓
references/vendor/
        ↓
Project interpretation and verified documentation
        ↓
docs/
        ↓
Independent controller implementation
```

Vendor reference files are not the project architecture itself. They provide the known-good physical baseline against which controller integration, sensing, timing, commutation, and control behavior are developed and verified.

## Redistribution

Only material that is appropriate for redistribution should be committed to this public repository. Vendor firmware archives and third-party source packages should be reviewed for licensing and redistribution terms before inclusion.
