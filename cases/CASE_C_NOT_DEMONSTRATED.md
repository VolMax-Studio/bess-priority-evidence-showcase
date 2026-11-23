# Case Study C: Telemetry Dropout & Indeterminate Records (`NOT_DEMONSTRATED`)

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
| **Market Dispatch Request** | `4.00 MW` continuous for `30.0 min` | Total requested energy: `2.0000 MWh` |
| **Reserve Margin Exhaustion** | **Minute `12.0`** | Point in time where `0.80 MWh` is fully discharged at `4.00 MW` |

Case C evaluates a synthetic reference trace modeling a telemetry dropout (such as a SCADA gateway disconnect or communication timeout) across $t = 11.0 \dots 16.0\text{ min}$, directly straddling the minute-12 reserve exhaustion boundary.

---

## 2. Chronological Evidence Sequence & Dual-Metric Accounting

```
Time Window      Market Request  EMS Setpoint  PCC Meter  BMS SoC       Status / Accounting
0.0 - 10.0 min   4.00 MW         4.00 MW       4.00 MW    38.0->31.33%  10 both-ends valid intervals [0, 10): 0.6667 MWh
10.0 - 11.0 min  4.00 MW         4.00 MW*      4.00 MW*   [t=11 MISS]   Start-valid only [10, 11): +0.0667 MWh (0.7333 MWh model sum)
11.0 - 17.0 min  4.00 MW         [MISSING]     [MISSING]  [MISSING]     TELEMETRY BLACKOUT (6 intervals [11, 17); 0.4000 MWh market req.)
17.0 - 30.0 min  4.00 MW         0.00 MW       0.00 MW    30.00%        13 both-ends valid intervals [17, 30): 0.8667 MWh curtailed
```

### Distinguishing Sample Counts, Start-Sample Intervals, and Validated-Endpoint Intervals

Under the S0 verification contract, a 1-minute interval $[t, t+1)$ is counted as **both-ends validated** only when telemetry is valid at **both** its start ($t$) and end ($t+1$). In Case C, sample $t = 10.0\text{ min}$ is valid, while sample $t = 11.0\text{ min}$ is missing. Because of this boundary transition, three distinct coverage metrics must be kept separate:

1. **Valid Discrete Samples (`25 / 31` samples · `80.6%`):**  
   11 pre-blackout samples ($t = 0 \dots 10\text{ min}$) + 14 post-blackout samples ($t = 17 \dots 30\text{ min}$). Samples $t = 11 \dots 16\text{ min}$ are missing (`MISSING_GAP`).
2. **Fully Validated Adjacent-Sample Intervals (`23 / 30` intervals · `76.7%`):**  
   Intervals where **both** adjacent endpoints are valid:
   - Pre-blackout: 10 intervals $[0, 1), \dots, [9, 10)\text{ min}$ $\implies 10 \times \frac{4.00\text{ MW}}{60\text{ min}} = \mathbf{0.6667\text{ MWh}}$ of modeled discharge integrated across both-ends validated intervals under the idealized piecewise-constant model, matching the recorded instantaneous SoC drop from $38.00\%$ to $31.33\%$ ($-0.6667\text{ MWh}$).
   - Post-blackout: 13 intervals $[17, 18), \dots, [29, 30)\text{ min}$ $\implies 13 \times \frac{4.00\text{ MW}}{60\text{ min}} = \mathbf{0.8667\text{ MWh}}$ of documented curtailed market request across both-ends validated intervals at $0.00\text{ MW}$.
   - **Interval `[10, 11)` is excluded** from `valid_interval_count` (`23 / 30`) because its closing endpoint at $t = 11.0\text{ min}$ is missing.
3. **Start-Sample Accounting Intervals (`24 / 30` intervals · `80.0%`):**  
   When the piecewise-constant model projects each valid start sample $t$ forward across $[t, t+1)$, sample $t = 10.0\text{ min}$ contributes $+0.0667\text{ MWh}$ for `[10, 11)`, yielding a start-sample accounting sum of **`0.7333 MWh`** (`0.73 MWh`). Because `[10, 11)` lacks a valid closing endpoint at $t = 11.0\text{ min}$, `0.7333 MWh` is a **start-sample model accounting figure**, distinct from the `0.6667 MWh` integrated across intervals with two validated endpoints.

### Market-Request Energy Allocation (Partition of Requested Market Energy)

Under start-sample accounting, the `2.0000 MWh` total market dispatch request partitions as:
- **Start-sample pre-blackout accounting (`[0, 11)` min, 11 intervals):** `0.7333 MWh` (`0.6667 MWh` across 10 both-ends valid intervals + `0.0667 MWh` across start-only interval `[10, 11)`)
- **Telemetry blackout market request (`[11, 17)` min, 6 intervals):** `0.4000 MWh` *(known requested market energy; **actual discharge during this gap is unobserved**)*
- **Post-blackout curtailed market request (`[17, 30)` min, 13 intervals):** `0.8667 MWh`
- **Market-request allocation sum:** $0.7333 + 0.4000 + 0.8667 = \mathbf{2.0000\text{ MWh}}$

> **Important Epistemic Distinction:** This equality represents a **market-request energy allocation** (partition of requested market energy under start-sample accounting)—it is **not** a proof of physical energy conservation, because battery behavior during the `0.4000 MWh` telemetry gap (`t = 11..16 min`) is unobserved.

---

## 3. Layered Evidence Evaluation

```
[ Market Request: 4.0 MW (30 min) ]
                |
                v
  [ EMS Command Layer ] ---------> INDETERMINATE (Missing setpoint across t = 11..16 min)
                |
                v
  [ Physical Measurement Layer ] -> INDETERMINATE (Missing meter & SoC across t = 11..16 min)
                |
                v
  [ Telemetry Completeness ] ----> FAILED / GAP (25/31 samples; 23/30 both-ends valid intervals)
                |
                v
  ===> PRIORITY CONTRACT VERDICT: NOT_DEMONSTRATED
```

1. **EMS Command Layer (`INDETERMINATE`):**  
   The record cannot establish whether a curtailment setpoint was issued at minute 12.0, delayed until minute 15.0, or omitted altogether during the blackout.
2. **Physical Measurement Layer (`INDETERMINATE`):**  
   Although SoC reports $30.00\%$ when telemetry resumes at minute 17.0, no records show what occurred during the missing interval:
   - Did the battery step to $0.00\text{ MW}$ at minute 12.0 and hold $30.00\%$?
   - Or did it discharge below $30.00\%$ and recharge during the unrecorded interval?
   - Or did an external protection relay trip the inverter?
3. **Telemetry Completeness (`INDETERMINATE`):**  
   Both the missing sample window ($t = 11 \dots 16\text{ min}$) and the unclosed interval `[10, 11)` leave the contractual decision boundary ($t = 12.0\text{ min}$) unobserved.

---

## 4. Why Neither `SUPPORTED` Nor `REFUTED` Is Defensible

In an assurance audit, **absence of evidence cannot be assumed to mean compliance or violation**:

- **Why Not `SUPPORTED`?**  
  Observing $30.00\%$ SoC at minute 17.0 does not establish what happened across $t = 11 \dots 16\text{ min}$. Compliance cannot be credited over unrecorded conflict intervals.
- **Why Not `REFUTED`?**  
  Conversely, a breach verdict requires affirmative records establishing a command or physical violation, which do not exist across the missing window.
- **The Only Defensible Engineering Verdict:**  
  Because available records establish neither compliance nor violation, the priority-contract verdict is strictly **`NOT_DEMONSTRATED`**.

---

## 5. Practical FAT/SAT Questions Raised by Case C

When evaluating telemetry resilience during commissioning or asset performance monitoring:
1. *Does the plant edge controller or RTU maintain a persistent non-volatile circular buffer for SCADA records during network disconnects?*
2. *When communication resumes, does the gateway backfill missing intervals into the historian with original hardware timestamps?*
3. *If communication drops between the EMS and inverter, what fail-safe mode governs active power (hold last setpoint, ramp to 0 MW, or trip)?*
4. *Can the asset owner substantiate compliance during a dispute if telemetry dropouts coincide with reserve transitions?*
