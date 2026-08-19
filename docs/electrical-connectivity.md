# Electrical Connectivity Baseline

This document defines the project-maintained electrical connectivity baseline extracted from the vendor-validated, known-good motor-control hardware.

The original vendor board schematic and board documentation are the initial hardware reference sources. Project KiCad schematics and exported README figures are maintained documentation artifacts derived from that verified hardware baseline.

The intent is to separate **electrical correctness** from presentation. Any reconstructed KiCad schematic must first be checked against the original hardware documentation before it is treated as a maintained project connectivity reference.

## Three-Phase Power Stage

The inverter is a three-phase, six-switch topology with three electrically independent phase nodes.

### Phase A

```text
VM -> QHA.D
QHA.S -> MA
MA -> QLA.D
QLA.S -> R42
R42 -> GND
```

Gate-drive net:

```text
HOA -> QHA.G
LOA -> QLA.G
```

### Phase B

```text
VM -> QHB.D
QHB.S -> MB
MB -> QLB.D
QLB.S -> R43
R43 -> GND
```

Gate-drive net:

```text
HOB -> QHB.G
LOB -> QLB.G
```

### Phase C

```text
VM -> QHC.D
QHC.S -> MC
MC -> QLC.D
QLC.S -> R44
R44 -> GND
```

Gate-drive net:

```text
HOC -> QHC.G
LOC -> QLC.G
```

### Required invariants

- `MA`, `MB`, and `MC` must remain electrically independent.
- No phase midpoint may be shorted to another phase midpoint.
- Each low-side current shunt remains in the corresponding phase low-side return path.
- Gate-driver outputs must terminate on the intended MOSFET gate pins rather than at an unlabeled drawing boundary.

## Phase-Current Sensing

Each INA181A1 channel measures the differential voltage across its corresponding low-side shunt.

The following polarity assignments are documentation targets and must remain consistent with the original board schematic when reconstructed in KiCad.

### Phase A

```text
R42 high-side node -> INA181A1_A IN+
R42 low-side node  -> INA181A1_A IN-
INA181A1_A OUT     -> ISA
REF1V65            -> INA181A1_A REF
```

### Phase B

```text
R43 high-side node -> INA181A1_B IN+
R43 low-side node  -> INA181A1_B IN-
INA181A1_B OUT     -> ISB
REF1V65            -> INA181A1_B REF
```

### Phase C

```text
R44 high-side node -> INA181A1_C IN+
R44 low-side node  -> INA181A1_C IN-
INA181A1_C OUT     -> ISC
REF1V65            -> INA181A1_C REF
```

Common reference:

```text
REF1V65 = approximately 1.65 V
```

### Required invariants

- INA181A1 inputs must be shown across the shunt, not in series with the phase-current path.
- `REF1V65` must connect to the `REF` pin of all three current-sense amplifiers.
- `ISA`, `ISB`, and `ISC` are amplifier outputs routed to controller ADC inputs.
- `IN+` / `IN-` polarity in reconstructed documentation must match the original board schematic.

## Hall Interface

```text
HALLA -> controller GPIO
HALLB -> controller GPIO
HALLC -> controller GPIO
V_HALL -> Hall sensor supply
GND -> Hall sensor ground
```

`V_HALL` is a documentation-level functional name for the Hall-sensor supply. On the initial controller-board implementation, the Hall interface is supplied from +5 V; future controller implementations may expose the same function differently.

## Back-EMF Sensing

The three phase-derived back-EMF channels are independent.

```text
MA -> EMFA -> LM339 comparator channel A -> EOA
MB -> EMFB -> LM339 comparator channel B -> EOB
MC -> EMFC -> LM339 comparator channel C -> EOC
```

Each comparator channel compares its phase-derived signal against the board's common virtual-neutral/reference signal, `VN`.

Conceptually:

```text
EMFA -> comparator A input
VN   -> comparator A reference input
comparator A output -> EOA

EMFB -> comparator B input
VN   -> comparator B reference input
comparator B output -> EOB

EMFC -> comparator C input
VN   -> comparator C reference input
comparator C output -> EOC
```

### Required invariants

- Do not represent the LM339 as a single three-input comparator.
- Use three independent comparator channels from the LM339 package.
- Comparator polarity (`+` / `-`) must match the actual board schematic before the KiCad source is finalized.
- LM339 outputs are open-collector; any required pull-up network should be represented according to the actual hardware schematic.

## DC-Bus Voltage Sensing

```text
VM -> R_TOP -> VAD -> R_BOTTOM -> GND
```

More explicitly:

```text
VM
 |
R_TOP
 |
 +------> VAD -> controller ADC
 |
R_BOTTOM
 |
GND
```

### Required invariants

- `R_TOP` and `R_BOTTOM` must form one continuous divider between `VM` and `GND`.
- `VAD` is the divider midpoint.
- The exported figure must not show either divider resistor disconnected from the midpoint or ground.

## Schematic / Figure Workflow

1. Start from the original vendor board schematic and board documentation.
2. Reconstruct or simplify the circuit in KiCad using standard schematic symbols where useful for project documentation.
3. Verify pin-level connectivity, polarity, junctions, and functional net mapping against the original hardware reference.
4. Run ERC where applicable.
5. Export the verified schematic or simplified documentation view to SVG.
6. Use the SVG in the README or detailed documentation.

The README image and reconstructed KiCad schematic must not supersede the original hardware reference unless the project intentionally creates and validates a new hardware revision.
