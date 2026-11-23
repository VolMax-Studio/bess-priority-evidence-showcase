# Engineering Brief: BESS Priority-Contract Verification

> **`SYNTHETIC — ILLUSTRATIVE, NOT FIELD VALIDATED`**  
> Copyright © 2026 VolMax Studio. All rights reserved. No open-source license is granted.

---

## 1. The Operational Problem: Multi-Use Asset Conflicts

Industrial Battery Energy Storage Systems (BESS) are increasingly deployed as multi-use assets. A common configuration pairs:
1. **Behind-the-Meter (BTM) Site Protection:** Maintaining a mandatory reserve capacity (e.g. 30.0% SoC) dedicated to facility backup, critical power continuity, or demand peak-shaving.
2. **Front-of-the-Meter (FTM) Market Participation:** Responding to external dispatch requests (e.g. wholesale arbitrage, frequency response, or manual capacity dispatch).

When an external market dispatch request coincides with low available battery energy, the two obligations enter a direct physical conflict:
- **If dispatch follows the market request past the floor:** The battery drains into the site reserve, risking facility exposure or violating site backup covenants.
- **If dispatch protects the site reserve:** Market export must stop at the reserve floor, creating curtailed market demand that must be auditable from operational records.

---

## 2. Mathematical Formulation & Conflict Geometry

To analyze this interaction cleanly, S0 defines a canonical synthetic reference scenario under an idealized, lossless piecewise-constant 1-minute model (evaluating telemetry records against the contract rather than executing a live EMS controller):

### System Parameters
- **Assumed Nominal Usable Capacity ($E_{\text{nom}}$):** $10.00\text{ MWh}$
- **Initial State of Charge ($\text{SoC}_0$):** $38.00\%$ $\implies E_0 = 3.80\text{ MWh}$
- **Mandatory Site Reserve Floor ($\text{SoC}_{\text{floor}}$):** $30.00\%$ $\implies E_{\text{floor}} = 3.00\text{ MWh}$
- **Available Usable Energy Margin ($\Delta E_{\text{margin}}$):**
  $$\Delta E_{\text{margin}} = E_0 - E_{\text{floor}} = 3.80\text{ MWh} - 3.00\text{ MWh} = \mathbf{0.80\text{ MWh}}$$
- **External Market Dispatch Request ($P_{\text{req}}$):** $4.00\text{ MW}$ continuous export for $\Delta T = 30.0\text{ min}$ ($0.50\text{ h}$)
- **Total Requested Market Energy ($E_{\text{req}}$):**
  $$E_{\text{req}} = P_{\text{req}} \times \Delta T = 4.00\text{ MW} \times 0.50\text{ h} = \mathbf{2.00\text{ MWh}}$$

### Boundary Condition at Minute 12.0
Under continuous $4.00\text{ MW}$ discharge, the available energy margin $\Delta E_{\text{margin}}$ is exhausted at time $t^*$:
$$t^* = \frac{\Delta E_{\text{margin}}}{P_{\text{req}}} = \frac{0.80\text{ MWh}}{4.00\text{ MW}} = 0.20\text{ hours} = \mathbf{12.0\text{ minutes}}$$

```
Energy (MWh)
  4.0 |  [Initial: 3.80 MWh / 38%]
      |     \
  3.0 |======\============================================= [RESERVE FLOOR: 3.00 MWh / 30%]
      |       \ (Case B: breaches floor)
  2.0 |        \
      |         \
  1.0 |          \---> [Case B Final: 1.80 MWh / 18%]
      |
  0.0 +--------------------+-------------------------------+
     t=0                  t=12                           t=30 (min)
              Phase 1                   Phase 2
         (Usable Margin)           (Curtailed Window)
```

### Market-Request Energy Allocation Under Priority Compliance
- **Modeled Delivered Market Energy ($t = 0 \dots 12\text{ min}$):**  
  $$E_{\text{delivered}} = 4.00\text{ MW} \times \frac{12}{60}\text{ h} = \mathbf{0.80\text{ MWh}}$$
- **Curtailed Market Request ($t = 12 \dots 30\text{ min}$):**  
  $$E_{\text{curtailed}} = 4.00\text{ MW} \times \frac{18}{60}\text{ h} = \mathbf{1.20\text{ MWh}}$$
- **Market-Request Energy Allocation:**  
  $$E_{\text{delivered}} + E_{\text{curtailed}} = 0.80 + 1.20 = 2.00\text{ MWh} = E_{\text{req}}$$

---

## 3. Why Casual Log Inspection Fails

Asset owners and technical advisors often rely on high-level spreadsheet summaries or monthly billing meters to evaluate compliance. In practice, casual log inspection fails for three reasons:

1. **Equating Setpoints with Physical Action:**  
   An EMS log showing a $0.0\text{ MW}$ command does not prove the battery stopped discharging. Inverter Modbus communication timeouts, frozen registers, or faulty relays can leave physical export running unchecked.
2. **Equating Physical Inactivity with Command Compliance:**  
   If the battery stops discharging at minute 12 because an over-current breaker tripped or an external protection relay opened while the EMS continued issuing discharge setpoints, the physical reserve was preserved—but the command layer still violated its priority obligation.
3. **Ignoring Telemetry Dropouts During Transitions:**  
   Telemetry networks can drop packets during switching transients or gateway resets. If records are missing across the 12-minute transition (as in Case C), inferring compliance from pre- and post-event values is an unsubstantiated assumption.

---

## 4. Practical FAT/SAT Commissioning Checklist

For commissioning engineers conducting Factory Acceptance Testing (FAT) or Site Acceptance Testing (SAT) on utility or industrial BESS projects:

- [ ] **Dual-Layer Logging:** Ensure time-synchronized logging ($1\text{ Hz}$ or $1\text{-minute}$ minimum) records both the EMS setpoint register and the PCC power meter separately.
- [ ] **Reserve-Floor Curtailment Test:** Inject a test market dispatch request as the battery approaches the reserve floor during FAT/SAT, and verify from separate command and meter logs that dispatch curtailment triggers via priority logic before hard BMS protection disconnects.
- [ ] **Data Completeness Validation:** Inspect gateway buffer persistence during network disconnects. Verify that timestamps are synchronized via NTP/PTP and that no sample gaps occur across transition boundaries.
- [ ] **Deterministic Adjudication Criteria:** Define quantitative pass/fail rules in the commissioning protocol rather than relying on qualitative visual chart inspection.
