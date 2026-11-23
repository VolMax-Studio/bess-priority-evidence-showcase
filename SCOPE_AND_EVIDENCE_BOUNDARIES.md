# Scope & Evidence Boundaries — `Engineering Casebook` (Showcase v1.1)

> **`SYNTHETIC — ILLUSTRATIVE, NOT FIELD VALIDATED`**  
> Copyright © 2026 VolMax Studio. All rights reserved. No open-source license is granted.

---

## 1. What This Showcase Demonstrates vs. What Is Explicitly Excluded

| Aspect | Included in Engineering Casebook | Explicitly Not Claimed / Excluded |
| :--- | :--- | :--- |
| **Data Origin** | Deterministic synthetic time-series traces under a predefined S0 contract (`10.0 MWh` assumed nominal usable capacity, `38.0% -> 30.0%` floor, `4.0 MW` market request over `30 min`) | No live EMS controller execution, real plant hardware, client SCADA/EMS/BMS telemetry, field incident, or named vendor bug |
| **Physical Model** | Idealized lossless 1-minute reference accounting to clarify the conflict geometry at `t = 12 min` | Inverter efficiency curves, auxiliary loads, cell degradation, thermal derating, or sub-second transients |
| **Evidence Layers** | Explicit separation of `Market Request`, `Command Layer` (`EMS Setpoint`), `Physical Power Layer` (`PCC Meter` + `BMS SoC`), and `Telemetry Completeness` | Issued EMS setpoints never substitute for metered physical compliance; missing data cannot become `SUPPORTED` |
| **Meaning of `REFUTED`** | Violation of the contracted priority rule (either via violating commands, physical reserve depletion, or both) | Does not assert physical over-discharge when a violation originates purely in the command layer (`Command: VIOLATED`, `Physical: COMPLIANT`) |
| **Audit & Independence** | Results originate from deterministic internal verification software separated from data generation and reviewed through internal read-only engineering reviews | Not an external third-party audit/certification, not an industrial accreditation, and not open-source public self-reproduction |
| **Intellectual Property** | Visual casebook outcomes, analytical case studies, and bounded engineering briefs | Proprietary adjudication code, exact numeric tolerances, negative-control suites, and dataset generators are excluded and not licensed for reuse |

---

## 2. Three Contract Outcomes at a Glance

1. **`SUPPORTED` ([Case Study A](cases/CASE_A_SUPPORTED.md)):** Complete, mutually consistent records where both the command layer (`COMPLIANT`) and measured physical layer (`COMPLIANT`) honor the contracted reserve floor across the entire horizon.
2. **`REFUTED` ([Case Study B](cases/CASE_B_REFUTED.md)):** Available records affirmatively demonstrate a breach of the contracted priority rule in the EMS command layer, the physical measurement/energy-balance layer, or both.
3. **`NOT_DEMONSTRATED` ([Case Study C](cases/CASE_C_NOT_DEMONSTRATED.md)):** Missing, stale, unsynchronized, or physically inconsistent evidence prevents establishing compliance or breach across the critical conflict interval.

---

## 3. Licensing & Reuse Notice

All files in this showcase package (text, specifications, case studies, and figures) are **Copyright © 2026 VolMax Studio. All rights reserved.** No open-source license is granted.
