# Case Study A: Compliant Dispatch Curtailment (`SUPPORTED`)

> **`SYNTHETIC — ILLUSTRATIVE, NOT FIELD VALIDATED`**  
> Copyright © 2026 VolMax Studio. All rights reserved. No open-source license is granted.

---

## 1. Scenario Summary

| Parameter | Specification | Physical Meaning |
| :--- | :--- | :--- |
| **Assumed Nominal Usable Capacity** | `10.0 MWh` | Assumed usable reference energy capacity (S0 contract) |
| **Initial State of Charge (SoC)** | `38.00%` (`3.80 MWh`) | Usable starting energy |
| **Mandatory Site Reserve Floor** | `30.00%` (`3.00 MWh`) | Non-negotiable industrial reserve floor |
| **Usable Energy Margin** | `0.80 MWh` (`+8.00% SoC`) | Energy available for market discharge above reserve floor |
| **Market Dispatch Request** | `4.00 MW` continuous for `30.0 min` | Total requested energy: `2.00 MWh` |
| **Reserve Margin Exhaustion** | **Minute `12.0`** | Point in time where `0.80 MWh` is fully discharged at `4.00 MW` |

Case A evaluates a synthetic reference telemetry trace depicting compliant priority behavior: the recorded EMS command stream follows the market request while usable margin remains above the reserve floor and steps to zero at the reserve boundary (this demonstration evaluates generated synthetic records and does not execute a live operational EMS controller).

---

## 2. Chronological Evidence Sequence

```
Time Interval    Market Request    EMS Setpoint    PCC Meter    BMS SoC         Recorded Trace Behavior
0.0 - 12.0 min   4.00 MW           4.00 MW         4.00 MW      38.0% -> 30.0%  Discharging usable margin (0.80 MWh)
At t = 12.0 min  4.00 MW           4.00 -> 0.00 MW 4.00 -> 0.00 MW 30.00%       Recorded setpoint & meter step to 0.00 MW
12.0 - 30.0 min  4.00 MW           0.00 MW         0.00 MW      30.00% (flat)   Market curtailed (1.20 MWh); reserve intact
```

- **Phase 1 ($t = 0.0 \dots 12.0\text{ min}$):**  
  The synthetic trace records a $4.00\text{ MW}$ EMS discharge setpoint and $4.00\text{ MW}$ export at the Point of Common Coupling (PCC) meter. Under the idealized piecewise-constant model, BMS SoC decreases linearly from $38.00\%$ to $30.00\%$, integrating $0.80\text{ MWh}$ by minute $12.0$ ($\frac{12}{60}\text{ h} \times 4.00\text{ MW} = 0.80\text{ MWh}$).
- **Phase 2 ($t = 12.0 \dots 30.0\text{ min}$):**  
  At minute $12.0$ ($30.00\%$ reserve floor), the synthetic EMS setpoint trace steps discretely from $4.00\text{ MW}$ to $0.00\text{ MW}$. Recorded active power drops to $0.00\text{ MW}$, and BMS SoC holds strictly at $30.00\%$ across the remaining 18 minutes.
- **Market-Request Energy Allocation:**  
  - Total market requested: `2.00 MWh`
  - Modeled delivered export energy: `0.80 MWh`
  - Curtailed market request: `1.20 MWh`

---

## 3. Layered Evidence Evaluation

An engineering audit requires evaluating the command layer separately from the physical measurement layer:

```
[ Market Request: 4.0 MW (30 min) ]
                |
                v
  [ EMS Command Layer ] ---------> COMPLIANT (Steps to 0.0 MW at t = 12.0 min)
                |
                v
  [ Physical Measurement Layer ] -> COMPLIANT (Meter drops to 0.0 MW; SoC holds at 30.0%)
                |
                v
  [ Telemetry Completeness ] ----> VALID (31 / 31 valid 1-minute samples; 30 / 30 valid intervals)
                |
                v
  ===> PRIORITY CONTRACT VERDICT: SUPPORTED
```

1. **EMS Command Layer (`COMPLIANT`):**  
   The recorded setpoint matches the market request during the unconstrained window ($0 \dots 12\text{ min}$) and steps to $0.00\text{ MW}$ at minute $12.0$. No discharge command is recorded while SoC is at or below the $30.0\%$ floor.
2. **Physical Measurement Layer (`COMPLIANT`):**  
   Recorded active power tracks the commanded setpoint. Modeled step-integrated energy matches the BMS SoC trajectory, and the recorded SoC does not breach the $30.00\%$ reserve threshold.
3. **Telemetry Completeness (`VALID`):**  
   All 31 expected 1-minute telemetry records (`30 / 30` adjacent intervals) are present, valid, and synchronized.

---

## 4. Why This Outcome Is `SUPPORTED`

The priority contract requires that **both** the command-layer record and the physical measurement-layer record satisfy contract constraints across complete, unbroken telemetry:
- A compliant setpoint record alone without metered verification is insufficient.
- Physical reserve preservation that occurs due to external protective trips while an EMS setpoint continues commanding discharge is not compliant control.
- In Case A, both recorded layers affirmatively align with the priority contract across the entire 30-minute decision horizon. Therefore, the priority-contract verdict is **`SUPPORTED`**.

---

## 5. Practical FAT/SAT Questions Raised by Case A

When witnessing or reviewing an operational BESS during Factory or Site Acceptance Testing:
1. *Does the operational EMS curtail dispatch smoothly via ramp-down or via an abrupt step/trip?*
2. *Is curtailment triggered by live BMS SoC telemetry or by internal controller state estimation?*
3. *What deadband or hysteresis is configured around the 30.0% boundary to prevent rapid cycling under fluctuating cell voltages?*
4. *Are curtailment transitions explicitly tagged in the SCADA event historian, or do they appear as unlabelled setpoint drops?*
