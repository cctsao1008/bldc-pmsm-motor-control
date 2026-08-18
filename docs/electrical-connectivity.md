# Electrical Connectivity Baseline

This document defines the electrical connectivity that must be preserved by future KiCad schematics and exported README figures.

The intent is to separate **electrical correctness** from presentation. The KiCad schematic should be treated as the source of truth; README SVGs are presentation artifacts derived from the verified schematic.

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

## Hall Interface

```text
HALLA -> controller GPIO
HALLB -> controller GPIO
HALLC -> controller GPIO
V_HALL -> Hall sensor supply
GND -> Hall sensor ground
```

Where the Hall supply is configured as 5 V or 3.3 V, the schematic should keep `V_HALL` as the functional net name and annotate the configured voltage separately.

## Back-EMF Sensing

The three phase-derived back-EMF channels are independent.

```text
MA -> EMFA -> LM339 comparator channel A -> EOA
MB -> EMFB -> LM339 comparator channel B -> EOB
MC -> EMFC -> LM339 comparator channel C -> EOC
```

Each comparator channel compares its phase-derived signal against the board's common zero-cross reference / virtual-neutral reference.

Conceptually:

```text
EMFA -> comparator A input
VREF/VN -> comparator A reference input
comparator A output -> EOA

EMFB -> comparator B input
VREF/VN -> comparator B reference input
comparator B output -> EOB

EMFC -> comparator C input
VREF/VN -> comparator C reference input
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

1. Build the circuit in KiCad using standard schematic symbols.
2. Verify pin-level connectivity and junctions.
3. Run ERC where applicable.
4. Review the schematic against this connectivity baseline and the original board schematic.
5. Export the verified schematic or simplified documentation view to SVG.
6. Use the SVG in the README.

The README image should never become the source of truth for electrical connectivity.
