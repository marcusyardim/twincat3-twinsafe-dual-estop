\# TwinCAT 3 TwinSAFE Dual E-Stop Template



A modular Beckhoff TwinCAT 3 and TwinSAFE template featuring a dual-channel E-Stop safety group (`FBEstop1`), a PLC interface program (`PRG\_SafetyInterface`) with single-scan rising-edge (`R\_TRIG`) pulse logic, and a local HMI simulation (`VISU\_SafetySim`) for testing without physical hardware\[cite: 20, 21, 22].



\## Project Structure



\- \*\*PLC (`PlcSafety`)\*\*\[cite: 21]

&#x20; - `GVL\_Safety` — Command, status, and safety alias variables\[cite: 21].

&#x20; - `PRG\_SafetyInterface` — Generates single-scan `R\_TRIG` pulses for reset and error acknowledgment, derives `bEStop1\_Tripped` from `NOT bEStop1\_SafeHealthy`, and manages group run enable (`bSafety\_GroupRunOut`).

&#x20; - `MAIN` — Calls `PRG\_SafetyInterface()` each PLC scan.

&#x20; - `VISU\_SafetySim` — Operator-style simulation screen with momentary command buttons (\*\*Reset E-Stop\*\*, \*\*Ack Fault\*\*) and status indicators (\*\*EStop1 SafeHealthy\*\*, \*\*EStop1 Tripped\*\*, \*\*Safety ComFault\*\*, \*\*RestartOut (Pulse)\*\*, \*\*ErrAckOut (Pulse)\*\*)\[cite: 20, 21].

\- \*\*Safety (`TwinSafeProject`)\*\*\[cite: 22]

&#x20; - `TwinSafeGroup1.sal` — Dual-channel E-Stop logic (`FBEstop1` with `bEStop1\_Ch1`, `bEStop1\_Ch2`, and `100 ms` delay)\[cite: 22].

&#x20; - `Alias Devices` — `.sds` alias mappings (`EStop1\_In.sds`, `EStop1\_Out.sds`, `EStop1\_Restart.sds`, `ErrorAcknowledgement.sds`, `EStop1\_SafeHealthy.sds`, `Run.sds`, `Safety\_ComFault.sds`)\[cite: 22].

\- \*\*I/O Tree\*\*

&#x20; - Includes reference EtherCAT / TwinSAFE device structure (`EK1100`, `EL6910`, `EL1904`, `EL2904`) for template completeness.



\## Key Signals (`GVL\_Safety`)



| Variable | Direction / Role | Purpose |

| :--- | :--- | :--- |

| `bEStop1\_ResetCmd` | HMI / Command (`BOOL`) | Momentary reset request (spring-return / tap) |

| `bSafety\_ErrAckCmd` | HMI / Command (`BOOL`) | Momentary safety fault acknowledge request |

| `bEStop1\_RestartOut` | PLC -> Safety (`%Q\*`) | Single-scan rising-edge restart pulse (`R\_TRIG`) |

| `bSafety\_ErrAckOut` | PLC -> Safety (`%Q\*`) | Single-scan rising-edge error ack pulse (`R\_TRIG`) |

| `bSafety\_GroupRunOut` | PLC -> Safety (`%Q\*`) | TwinSAFE group run enable signal |

| `bEStop1\_SafeHealthy` | Safety -> PLC (`%I\*`) | `TRUE` when E-Stop safe output is healthy |

| `bEStop1\_Tripped` | Derived Status (`BOOL`) | `TRUE` when `NOT bEStop1\_SafeHealthy` |

| `bSafety\_ComFault` | Safety -> PLC (`%I\*`) | Safety communication / group fault status |



\## Running in Simulation (No Physical Hardware)



This template is verified in \*\*offline / simulation mode\*\* without physical EtherCAT or TwinSAFE terminals connected:



1\. Open the solution in TwinCAT XAE (Visual Studio).

2\. Keep physical EtherCAT/I/O devices disabled if running locally without hardware, and verify the solution builds with \*\*0 Errors\*\*.

3\. Log in and start `PlcSafety`, then open `VISU\_SafetySim` to test momentary button presses, status lamps, and fault/trip transitions\[cite: 20, 21].



\## Deploying to Real Hardware



Before commissioning on physical machinery:



1\. Re-enable the EtherCAT master device and align `AmsNetId` / target process image settings with your controller and `EL6910`.

2\. Link `GVL\_Safety` `%I\*` and `%Q\*` variables and TwinSAFE aliases (`.sds`) to actual physical I/O and safety modules\[cite: 22].

3\. Perform full physical safety validation and commissioning in accordance with applicable machine safety standards.

