# SupervisoryControl

Supervisory controller synthesis for the **Hvide Sande storm surge barrier** using Eclipse ESCET (CIF).

## Overview

The Hvide Sande storm surge barrier separates the North Sea from Ringkobing Fjord on the west coast of Denmark. This project models the barrier as a discrete event system and synthesizes a maximally permissive supervisory controller that enforces safety and liveness requirements.

### Physical System

- **4 sluice gates** with 3 positions each (Closed/Halfway/Open)
- **Tidal sea level** cycling deterministically through 6 discrete levels (0-5m)
- **Fjord water level** affected by rain inflow and gate-controlled drainage
- **2 harbors** (North/South) with ship traffic and controllable entry signs
- **Gate actuation timer** enforcing cooldown between raise commands

### Requirements Enforced

| Req | Description |
|-----|-------------|
| **G1** | All gates must be closed during storm surge (sea level >= 5m) |
| **G2** | Gates at most halfway open when fjord level is normal (2-3m) |
| **G3** | Minimum interval between gate raise commands (timer cooldown) |
| **L1** | Fjord level must not exceed 4m (flood prevention) |
| **L2** | Fjord level must not fall below 1m (ecological minimum) |
| **S1** | Ships may only enter when water flow < 100 m3/s |
| **S2** | Ships must always eventually be able to pass (liveness) |

### Key Modeling Features

- **Turn-based clock**: guarantees the supervisor a reaction window after each tick
- **Storm warning**: supervisor gets advance notice before sea reaches maximum
- **Batch gate closure**: atomic `c_close_all` for emergency storm protection
- **Deterministic fjord dynamics**: fjord level changes are predictable consequences of gate positions

## Files

| File | Description |
|------|-------------|
| `plant.cif` | Plant model + requirements (input to synthesis) |
| `plant.cifcode` | Compiled CIF code (generated) |
| `plant.tooldef` | Automation script: compile check, simulation, synthesis |
| `supervisor.cif` | Synthesized supervisor (generated output from tooldef) |

## Usage

Requires [Eclipse ESCET](https://www.eclipse.org/escet/) v10.0.

```bash
# Compile check
cifsim plant.cif --compile-only=on

# Run synthesis
cifdatasynth plant.cif -o supervisor.cif

# Simulate the supervised system
cifsim supervisor.cif -i auto -a random -t 500 \
    -o "trans-always" --non-urgent-events="*"

# Or run everything via tooldef
tooldef plant.tooldef
```
