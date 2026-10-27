# Stage A - Rockwell Leg: Micro850 Simulator, Web SCADA Zone 02

## 1. Controller decision

**Micro850**, catalog family 2080-LC50 (48-point combo, e.g. `2080-LC50-48QWB-SIM`, matches the free CCW Simulator's I/O count). Not Micro870.

| | Micro850 | Micro870 |
|---|---|---|
| Free CCW Simulator support | Native — the simulator emulates a Micro850 | Not the simulator's target profile; higher mismatch risk |
| PowerFlex 520-series Class 1 EtherNet/IP | Yes (fw 21.011+) | Yes (fw 21.011+) |
| I/O / memory | Enough for this project | More than needed |

Decision logged in `docs/decisions-log.md`.

## 2. Project setup in CCW 23.0.0

1. **File → New Project.** Controller: Micro850, catalog `2080-LC50-48QWB _SIM` (or the 24-point variant if that's what the simulator wizard offers — match whatever the SIM target proposes).
2. Confirm the project shows the controller name ending in `SIM` — that's your simulator target, no hardware needed.
3. **Global Variables** window → paste the declarations from §3 below. Use Global (not Program) scope so they're visible for OPC UA/Ignition exposure later.
4. **Programs → Prg → Structured Text** (or add a new ST POU if `Prg` defaults to Ladder) → paste the code from §4.
5. Build (F7) and fix any syntax flags — CCW's ST editor is IEC 61131-3, close to what you've already written in CODESYS.

## 3. Global variable declarations
These variables are accessed through the Z02_ prefix or other global identifiers. The data types below are inferred from how the code uses them and the earlier declarations you supplied; verify them against your CCW Global Variables table. as a csv file
```iecst

No.	Variable	Suggested / known type	Purpose
1	Z02_START	   BOOL	   Start command
2	Z02_STOP	      BOOL	   Stop command
3	Z02_FAULT_RESET	BOOL	Fault reset command
4	Z02_AUTO	BOOL	Operator manual or auto selection (TRUE = auto)
5	Z02_SIM_ACTIVE	BOOL	Simulation initialization status
6	Z02_SIM_ENABLE	BOOL	Enables simulation
7	Z02_SIM_FLOW_TARGET	REAL	Target flow
8	Z02_SIM_FLOW_RATE	REAL	Flow ramp rate
9	Z02_SIM_MOISTURE_RATE	REAL	Moisture increase rate
10	Z02_SIM_DRY_RATE	REAL	Moisture decrease rate
11	Z02_SIM_LOW_FLOW_LIMIT	REAL	Low-flow threshold
12	Z02_MOISTURE_LOW_SP	REAL	Low moisture setpoint
13	Z02_MOISTURE_HIGH_SP	REAL	High moisture setpoint
14	Z02_IRR_SPD_SP	REAL	Irrigation pump speed setpoint
15	Z02_MOISTURE	REAL	Simulated soil moisture
16	Z02_SIM_COMM_FAULT	BOOL	Simulated communication fault
17	Z02_SIM_ACTUATOR_FAULT	BOOL	Simulated actuator fault
18	Z02_SIM_SENSOR_FAULT	BOOL	Simulated sensor fault
19	Z02_SIM_RESTRICTION	BOOL	Simulated flow restriction
20	Z02_FAULT	BOOL	Latched fault status
21	Z02_COMM_OK	BOOL	Communication status
22	Z02_PERMISSIVE	BOOL	Operating permissive
23	Z02_AUTO_ACTIVE	BOOL	AUTO mode active status
24	Z02_RUNNING	BOOL	Running status
25	Z02_COMMANDED	BOOL	Effective irrigation command
26	Z02_VALVE	REAL	Simulated valve position
27	Z02_FLOW	REAL	Simulated measured flow
28	Z02_PUMP_RUN	BOOL	Pump run command
29	Z02_PUMP_SPD	REAL	Pump speed command
30	Z02_PF_START	BOOL	PowerFlex start mapping
31	Z02_PF_STOP	BOOL	PowerFlex stop mapping
32	Z02_PF_SPDREF	REAL	PowerFlex speed reference
33	Z02_FLOW_OK	BOOL	Flow quality status
34	Z02_LOW_FLOW	         BOOL	Low-flow alarm status
35	Z02_PROCESS_OK         	BOOL	Overall process status
36	Z02_TOTAL_RUNTIME_SEC	DINT / UDINT	Accumulated runtime in seconds
37	Z02_RUN_FEEDBACK	      BOOL	Run feedback
38	Z02_MANUAL	            BOOL	Manual mode status
39	Z02_PF_RUNNING         	BOOL	PowerFlex running feedback
40	Z02_PF_FAULTED	         BOOL	PowerFlex fault status
41	rFlowTarget	REAL	Effective flow target after restriction logic
42	rFlowError	REAL	Difference between target flow and current flow
43	rFlowStep	REAL	Flow change per simulation update

global timer instance
SimTick	                  TON	Periodic simulation timer

Local variables

1   rMoistureStep	REAL	Moisture change per simulation update
2   rtStart	R_TRIG	Detects the rising edge of the start command
3   rtStop	R_TRIG	Detects the rising edge of the stop command
4   rtFaultReset	R_TRIG	Detects the rising edge of the fault reset command
5	 xAutoDemand	BOOL	Stores the AUTO moisture demand state
6	uiRuntimeTicks	UINT	Counts simulation ticks toward one runtime second

```

## 4. Structured Text program

```iecst
(* =============================================================
   ZONE 02 - MICRO850 IRRIGATION CONTROL
   Optimized CCW Implementation
   ============================================================= *)

(* 1. COMMAND EDGE DETECTION *)
rtStart(CLK := Z02_START);
rtStop(CLK := Z02_STOP);
rtFaultReset(CLK := Z02_FAULT_RESET);

(* =============================================================
   1.2 AUTOMATIC SIMULATION ENGINE INITIALIZATION 
   ============================================================= *)
IF NOT Z02_SIM_ACTIVE THEN
    Z02_SIM_ENABLE         := TRUE;
    Z02_SIM_FLOW_TARGET    := 100.0;
    Z02_SIM_FLOW_RATE      := 5.0;     (* Already configured correctly *)
    Z02_SIM_MOISTURE_RATE  := 0.05;    (* FIX: Overwriting the 0.0 state *)
    Z02_SIM_DRY_RATE       := 0.01;
    Z02_SIM_LOW_FLOW_LIMIT := 60.0;
    Z02_MOISTURE_LOW_SP    := 30.0;
    Z02_MOISTURE_HIGH_SP   := 70.0;
    Z02_IRR_SPD_SP         := 75.0;
    
    IF Z02_MOISTURE = 0.0 THEN
        Z02_MOISTURE       := 35.0;
    END_IF;
END_IF;

(* 2. SIMULATION AND COMMUNICATION STATUS *)
Z02_SIM_ACTIVE := Z02_SIM_ENABLE;
Z02_COMM_OK    := NOT Z02_SIM_COMM_FAULT;


(* 3. FAULT LATCH AND RESET *)
IF Z02_SIM_ACTUATOR_FAULT OR Z02_SIM_SENSOR_FAULT THEN
    Z02_FAULT := TRUE;
END_IF;

IF rtFaultReset.Q THEN
    IF NOT Z02_SIM_ACTUATOR_FAULT AND NOT Z02_SIM_SENSOR_FAULT THEN
        Z02_FAULT := FALSE;
    END_IF;
END_IF;


(* 4. PERMISSIVE *)
Z02_PERMISSIVE :=
    NOT Z02_FAULT
    AND NOT Z02_SIM_ACTUATOR_FAULT
    AND NOT Z02_SIM_SENSOR_FAULT
    AND Z02_COMM_OK
    AND Z02_SIM_ENABLE;


(* 5. AUTOMATIC MOISTURE DEMAND - HYSTERESIS *)
IF Z02_MOISTURE <= Z02_MOISTURE_LOW_SP THEN
    xAutoDemand := TRUE;
ELSIF Z02_MOISTURE >= Z02_MOISTURE_HIGH_SP THEN
    xAutoDemand := FALSE;
END_IF;


(* 6. RUNNING DECISION *)
IF Z02_PERMISSIVE THEN
    IF Z02_AUTO THEN
        Z02_AUTO_ACTIVE := TRUE;
        Z02_RUNNING     := xAutoDemand;
    ELSE
        Z02_AUTO_ACTIVE := FALSE;
        IF rtStart.Q THEN
            Z02_RUNNING := TRUE;
        END_IF;
    END_IF;
ELSE
    Z02_AUTO_ACTIVE := FALSE;
END_IF;

(* Manual/safety Stop always wins *)
IF rtStop.Q THEN
    Z02_RUNNING := FALSE;
    xAutoDemand := FALSE;
END_IF;


(* 7. SAFETY OVERRIDES *)
IF Z02_FAULT OR (NOT Z02_COMM_OK) OR (NOT Z02_SIM_ENABLE) THEN
    Z02_RUNNING := FALSE;
END_IF;


(* 8. PUMP, VALVE COMMANDS AND DRIVE REFERENCES *)
Z02_COMMANDED := Z02_RUNNING AND Z02_PERMISSIVE;
Z02_PUMP_RUN  := Z02_COMMANDED;

IF Z02_COMMANDED THEN
    Z02_PUMP_SPD := Z02_IRR_SPD_SP;
ELSE
    Z02_PUMP_SPD := 0.0;
END_IF;

Z02_PF_START  := Z02_PUMP_RUN;
Z02_PF_STOP   := NOT Z02_PUMP_RUN;
Z02_PF_SPDREF := Z02_PUMP_SPD;

(* =============================================================
   9. PERIODIC PROCESS SIMULATION (100MS TICK)
   ============================================================= *)

(* CRITICAL CCW FIX: The timer block must be executed UNCONDITIONALLY every single scan *)
SimTick(IN := NOT SimTick.Q, PT := T#100MS);

(* Only process the simulation updates on the exact scan the timer trips *)
IF SimTick.Q THEN

    (* 9.1 Valve position *)
    IF Z02_COMMANDED THEN
        Z02_VALVE := 100.0;
    ELSE
        Z02_VALVE := 0.0;
    END_IF;

    (* 9.2 Flow target, including restriction *)
    IF Z02_COMMANDED THEN
        rFlowTarget := Z02_SIM_FLOW_TARGET;
        IF Z02_SIM_RESTRICTION THEN
            rFlowTarget := rFlowTarget * 0.25;
        END_IF;
    ELSE
        rFlowTarget := 0.0;
    END_IF;

    (* 9.3 Flow ramp response *)
    rFlowError := rFlowTarget - Z02_FLOW;
    rFlowStep  := ABS(Z02_SIM_FLOW_RATE) * 0.1;
    
    IF rFlowStep < 0.1 THEN
        rFlowStep := 0.1;
    END_IF;

    IF rFlowError > rFlowStep THEN
        Z02_FLOW := Z02_FLOW + rFlowStep;
    ELSIF rFlowError < -rFlowStep THEN
        Z02_FLOW := Z02_FLOW - rFlowStep;
    ELSE
        Z02_FLOW := rFlowTarget;
    END_IF;

    IF Z02_FLOW < 0.0 THEN
        Z02_FLOW := 0.0;
    END_IF;

    IF Z02_SIM_SENSOR_FAULT THEN
        Z02_FLOW := 0.0;
    END_IF;

    (* 9.4 Moisture process response *)
    IF Z02_COMMANDED AND (Z02_FLOW > 0.0) THEN
        rMoistureStep := ABS(Z02_SIM_MOISTURE_RATE) * 0.1;
        Z02_MOISTURE  := Z02_MOISTURE + rMoistureStep;
    ELSE
        rMoistureStep := ABS(Z02_SIM_DRY_RATE) * 0.1;
        Z02_MOISTURE  := Z02_MOISTURE - rMoistureStep;
    END_IF;

    IF Z02_MOISTURE > 100.0 THEN Z02_MOISTURE := 100.0; END_IF;
    IF Z02_MOISTURE < 0.0   THEN Z02_MOISTURE := 0.0;   END_IF;

    (* 9.5 Flow quality and low-flow indication *)
    Z02_FLOW_OK    := TRUE;
    IF Z02_COMMANDED AND (Z02_FLOW < Z02_SIM_LOW_FLOW_LIMIT) THEN
        Z02_FLOW_OK := FALSE;
    END_IF;

    Z02_LOW_FLOW   := Z02_COMMANDED AND NOT Z02_FLOW_OK;
    Z02_PROCESS_OK := Z02_PERMISSIVE AND Z02_FLOW_OK;

    (* 9.6 Runtime accumulation *)
    IF Z02_RUNNING THEN
        uiRuntimeTicks := uiRuntimeTicks + 1;
        IF uiRuntimeTicks >= 10 THEN
            Z02_TOTAL_RUNTIME_SEC := Z02_TOTAL_RUNTIME_SEC + 1;
            uiRuntimeTicks := 0;
        END_IF;
    END_IF;

END_IF;



(* 10. OUTPUT AND STATUS MAPPING *)
Z02_RUN_FEEDBACK := Z02_RUNNING;
Z02_MANUAL       := NOT Z02_AUTO;
Z02_PF_RUNNING   := Z02_PUMP_RUN;
Z02_PF_FAULTED   := FALSE;


(* 11. MOMENTARY COMMAND CONSUMPTION *)
IF Z02_START THEN Z02_START := FALSE; END_IF;
IF Z02_STOP THEN Z02_STOP := FALSE; END_IF;
IF Z02_FAULT_RESET THEN Z02_FAULT_RESET := FALSE; END_IF;


```

## 5. Test it in the simulator

1. **Connect → Download** to the SIM target, then **Run mode**.
2. Open the **Watch window**, add `SystemState`, `LevelSensorPct`, `PumpRun`, `PumpSpeedRefPct`, `Valve1_Open/2/3`.
3. Force `StartCmd := TRUE` for one scan. Watch `SystemState` walk 0 → 10 → 20 → 21 → 22 → 30 → 0, with `LevelSensorPct` rising during FILL and falling during each irrigate/drain phase.
4. Remember the free simulator's 10-minute run-mode limit — if it stops, restart Run mode; variables reset to initialized values (expected, not a bug).

## 6. Connect Ignition via the native Micro800 driver

1. Gateway webpage → **Config → OPC UA → Device Connections → Create new Device**.
2. Choose the **Allen-Bradley Micro800 Driver**.
3. Hostname: `127.0.0.1` if Ignition and the simulator run on the same machine, otherwise that machine's LAN IP.
4. Leave port/slot at their Micro800 defaults — no Kepware/gateway step needed.
5. Save, confirm the device connection status goes green/"Connected." If not, check the simulator is in Run mode and the firewall isn't blocking the EtherNet/IP port.
6. **Designer → Tag Browser → OPC UA quick client** (or drag from the device browse tree) → pull in `StartCmd`, `StopCmd`, `ResetCmd`, `SystemState`, `LevelSensorPct`, `Valve1_Open/2/3`, `PumpRun`, `PumpSpeedRefPct`, fault bits.
7. Organize into a folder/UDT named consistently with your CODESYS tag structure (e.g. `Rockwell_IrrigationCell`) — keeps the eventual vendor comparison reading as one coherent system.
8. Bind a Perspective view: buttons for `StartCmd`/`StopCmd`/`ResetCmd`, a gauge for `LevelSensorPct`, indicators for each valve and `PumpRun`, a multi-state indicator for `SystemState`.

## 7. Repo note

This file is the text source-of-truth for the Rockwell leg, since CCW has no clean text-export path. Hand-update it after any logic changes made in the CCW GUI, per the version-control approach in `CONTRIBUTING.md`.
