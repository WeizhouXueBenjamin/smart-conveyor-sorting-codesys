# PLC Architecture

## System Overview

The project uses Factory I/O as the simulated machine and CODESYS Control Win as the PLC runtime.

Current control path:

Factory I/O sensors
→ PLC input mapping
→ internal feedback variables
→ sequence / machine logic
→ command variables
→ PLC output mapping
→ Factory I/O actuators

## I/O Boundary

Factory I/O variables are isolated at the PLC program boundary.

Inputs:
FIO.i... → xAt...

Outputs:
xCmd... → FIO.o...

Control logic does not directly read or write Factory I/O addresses.

## Sequence State Machine

The turntable sequence currently uses a single E_SequenceState variable:

SEQ_IDLE
→ SEQ_LOADING
→ SEQ_ROTATE_TO_UNLOAD
→ SEQ_DISCHARGING
→ SEQ_RETURN_TO_LOAD
→ SEQ_IDLE

Each state represents one stage of the turntable operating sequence.

## Control Structure

Current PLC variables are grouped into:

- Feedback
- Operating mode
- Operator requests
- Sequence state
- Machine status
- Internal machine state
- Permissives
- Fault status
- Actuator commands

Mode, operator requests, permissives and fault handling are currently architecture placeholders and will be implemented in later stages.

## Reference Baseline

The Factory I/O scene and original sample PLC logic came from the Factory I/O CODESYS OPC UA tutorial.

Project work completed so far includes:

- verification and correction of the I/O mapping
- separation of Factory I/O addresses from PLC control logic
- dedicated feedback and actuator command layers
- replacement of multiple sequence-state BOOL variables with a single state machine
- preparation of the PLC architecture for Manual/Auto, interlocks and fault handling
