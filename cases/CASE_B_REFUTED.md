# Case Study B: Priority Breach & Reserve Depletion (`REFUTED`)

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

Case B evaluates a synthetic reference telemetry trace modeling a priority-contract breach: the recorded EMS setpoint remains at `4.00 MW` past the minute-12 reserve boundary, and the modeled battery trace depletes into the protected reserve floor (this demonstration evaluates generated synthetic records and does not diagnose a fault in real third-party control software).

---

## 2. Chronological Evidence Sequence

```
Time Interval    Market Request    EMS Setpoint    PCC Meter    BMS SoC         Recorded Trace Behavior
0.0 - 12.0 min   4.00 MW           4.00 MW         4.00 MW      38.0% -> 30.0%  Discharging usable margin (0.80 MWh)
At t = 12.0 min  4.00 MW           4.00 MW         4.00 MW      30.00%          Reserve floor reached; NO CURTAILMENT
12.0 - 30.0 min  4.00 MW           4.00 MW         4.00 MW      30.0% -> 18.0%  Reserve floor breached (1.20 MWh below floor)
```

- **Phase 1 ($t = 0.0 \dots 12.0\text{ min}$):**  
  The synthetic trace records a $4.00\text{ MW}$ EMS discharge command and $4.00\text{ MW}$ PCC export, reaching $30.00\%$ SoC at minute $12.0$ ($0.80\text{ MWh}$ modeled discharge).
- **Phase 2 ($t = 12.0 \dots 30.0\text{ min}$):**  
  Past minute $12.0$, the synthetic EMS setpoint trace remains at $4.00\text{ MW}$ rather than stepping to $0.00\text{ MW}$. Modeled active power continues at $4.00\text{ MW}$ for an additional 18 minutes, integrating another $1.20\text{ MWh}$ ($\frac{18}{60}\text{ h} \times 4.00\text{ MW} = 1.20\text{ MWh}$).
- **Final Modeled State:**  
  At minute $30.0$, the recorded BMS SoC trace reaches **`18.00%`** (`1.80 MWh`), representing a **`12.0% SoC breach`** (`1.20 MWh` below the mandated `3.00 MWh` site reserve floor).
- **Market-Request Energy Allocation:**  
  - Total market requested: `2.00 MWh`
  - Modeled delivered export energy: `2.00 MWh`
  - Curtailed market request: `0.00 MWh`
  - Depletion below protected reserve floor: `1.20 MWh`

---

## 3. Layered Evidence Evaluation

```
[ Market Request: 4.0 MW (30 min) ]
                |
                v
  [ EMS Command Layer ] ---------> VIOLATED (Maintains 4.0 MW setpoint past t = 12.0 min)
                |
                v
  [ Physical Measurement Layer ] -> VIOLATED (Meter exports 4.0 MW; SoC depletes to 18.0%)
                |
                v
  [ Telemetry Completeness ] ----> VALID (31 / 31 valid 1-minute samples; 30 / 30 valid intervals)
                |
                v
  ===> PRIORITY CONTRACT VERDICT: REFUTED
```

1. **EMS Command Layer (`VIOLATED`):**  
   The synthetic command trace records $4.00\text{ MW}$ discharge while BMS SoC is at or below the $30.0\%$ reserve threshold across 18 consecutive minutes ($t = 12 \dots 30\text{ min}$).
2. **Physical Measurement Layer (`VIOLATED`):**  
   Recorded active power and BMS SoC traces establish depletion below the protected reserve floor under the S0 contract.
3. **Telemetry Completeness (`VALID`):**  
   All 31 expected records (`30 / 30` adjacent intervals) are present, valid, and synchronized.

---

## 4. Why This Outcome Is `REFUTED` (and Crucial Nuances of Breach)

The verdict **`REFUTED`** indicates that the evaluated records affirmatively establish a violation of the defined priority contract.

### The Essential Distinction Between Command Breach and Physical Depletion
In Case B, both the command-layer trace and the physical measurement-layer trace violate the priority contract. In industrial auditing practice, however, a priority-contract breach does **not** exclusively mean physical battery over-discharge:

- **Illustrative Command-Only Breach (`Command: VIOLATED`, `Physical: COMPLIANT`):**  
  Suppose a recorded EMS stream issues a $4.00\text{ MW}$ discharge command at $30.0\%$ SoC, while an external hardware interlock or battery protection unit (BPU) holds metered discharge at $0.00\text{ MW}$.  
  *Contract Verdict:* **`REFUTED`**. Why? Because the issued command record violated the contracted dispatch-priority rule; reserve preservation resulted from external hardware protection rather than compliant command governance.
- **Illustrative Physical-Only Breach (`Command: COMPLIANT`, `Physical: VIOLATED`):**  
  Suppose a recorded EMS stream steps to $0.00\text{ MW}$ at minute $12.0$, while metered power continues at $4.00\text{ MW}$ (e.g. due to frozen communication registers) and SoC drains below $30.0\%$.  
  *Contract Verdict:* **`REFUTED`**. A compliant setpoint command never substitutes for physical reserve protection.

In Case B, the synthetic records exhibit both violations simultaneously, yielding **`REFUTED`**.

---

## 5. Practical FAT/SAT Questions Raised by Case B

When evaluating reserve-protection behavior during commissioning or post-incident log review:
1. *Does an operational EMS treat an external market schedule as an unconditional setpoint rather than a constrained dispatch request?*
2. *Is the site reserve floor configured as a soft advisory target or an enforceable real-time constraint?*
3. *If SoC reaches the reserve boundary under high discharge, does the BMS assert dynamic discharge limits (DCL) ahead of hard breaker disconnect?*
4. *How quickly do site logs flag divergence when commanded power exceeds available energy margin?*
