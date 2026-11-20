# TwinCAT 3 TwinSAFE Dual E-Stop Template

A Beckhoff TwinCAT 3 and TwinSAFE template implementing a dual-channel E-Stop safety function (`FBEstop1`), a PLC interface routine (`PRG_SafetyInterface`) with rising-edge (`R_TRIG`) command conditioning, and a TwinCAT HMI/visualization simulation (`VISU_SafetySim`) for offline testing.

## Architecture Overview

- **Safety Logic (`TwinSafeProject`)**
  - `TwinSafeGroup1.sal`: Evaluates dual-channel E-Stop inputs (`bEStop1_Ch1`, `bEStop1_Ch2`) through `FBEstop1` with a `100 ms` discrepancy/delay setting.
  - `Alias Devices`: Defines `.sds` safety aliases (`EStop1_In.sds`, `EStop1_Out.sds`, `EStop1_Restart.sds`, `ErrorAcknowledgement.sds`, `EStop1_SafeHealthy.sds`, `Run.sds`, `Safety_ComFault.sds`).
- **PLC Interface (`PlcSafety`)**
  - `GVL_Safety`: Holds command, status, and process-image mapped safety variables (`%I*` / `%Q*`).
  - `PRG_SafetyInterface`: Uses `R_TRIG` blocks to convert momentary HMI/button commands into single-scan restart and error-acknowledgment pulses, derives `bEStop1_Tripped` from `NOT bEStop1_SafeHealthy`, and asserts `bSafety_GroupRunOut`.
  - `MAIN`: Executes `PRG_SafetyInterface()` on each PLC task cycle.
  - `VISU_SafetySim`: Provides momentary pushbuttons (**Reset E-Stop**, **Ack Fault**) and state indicators (**EStop1 SafeHealthy**, **EStop1 Tripped**, **Safety ComFault**, **RestartOut (Pulse)**, **ErrAckOut (Pulse)**).
- **I/O Reference Tree**
  - Includes a reference EtherCAT / TwinSAFE topology (`EK1100`, `EL6910`, `EL1904`, `EL2904`) for template structure.

## Key Signals (`GVL_Safety`)

| Signal | Type | Direction | Description |
| :--- | :--- | :--- | :--- |
| `bEStop1_ResetCmd` | `BOOL` | HMI / PLC | Momentary E-Stop reset request |
| `bSafety_ErrAckCmd` | `BOOL` | HMI / PLC | Momentary safety group error acknowledge request |
| `bEStop1_RestartOut` | `BOOL` | PLC -> Safety (`%Q*`) | Single-scan rising-edge restart pulse (`R_TRIG`) |
| `bSafety_ErrAckOut` | `BOOL` | PLC -> Safety (`%Q*`) | Single-scan rising-edge error ack pulse (`R_TRIG`) |
| `bSafety_GroupRunOut` | `BOOL` | PLC -> Safety (`%Q*`) | Safety group run enable signal |
| `bEStop1_SafeHealthy` | `BOOL` | Safety -> PLC (`%I*`) | `TRUE` when E-Stop safe output is healthy |
| `bEStop1_Tripped` | `BOOL` | Derived (`BOOL`) | `TRUE` when `NOT bEStop1_SafeHealthy` |
| `bSafety_ComFault` | `BOOL` | Safety -> PLC (`%I*`) | Safety communication or group fault indicator |

## Offline Simulation (Without Physical Hardware)

1. Open the solution in TwinCAT XAE (Visual Studio).
2. Keep physical EtherCAT/I/O devices disabled when running without hardware, and build the solution with **0 Errors**.
3. Activate/log in to `PlcSafety` and open `VISU_SafetySim` to verify momentary commands, edge pulses, trip/healthy transitions, and fault indication.

## Hardware Commissioning Notes

Before deploying to a physical machine:

1. Enable the EtherCAT master device and align target `AmsNetId` and process image settings with the controller and `EL6910`.
2. Link `GVL_Safety` `%I*` and `%Q*` signals and TwinSAFE `.sds` aliases to commissioned physical I/O.
3. Complete full safety validation and commissioning per applicable machine safety standards.