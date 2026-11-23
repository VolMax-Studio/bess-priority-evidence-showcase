# Evidence & Reproducibility Statement

> **`SYNTHETIC — ILLUSTRATIVE, NOT FIELD VALIDATED`**  
> Copyright © 2026 VolMax Studio. All rights reserved. No open-source license is granted.

---

## 1. Internal Reference Execution & Status

VolMax Studio designed, implemented, and verified the S0 BESS priority-contract assurance demonstration in a private reference environment:

- **Pipeline Architecture:**  
  The Python verification software is strictly decoupled from synthetic dataset generation. Generated input time-series traces are evaluated against deterministic step-wise energy-balance checks, command-tracking rules, telemetry-completeness constraints, and priority contracts (S0 evaluates telemetry traces and does not execute a live operational EMS controller).
- **Deterministic Checksum Baseline:**  
  In our private reference test environment, repeated end-to-end execution reproduces the internal baseline data tables and JSON verification reports byte-for-byte against archived SHA-256 manifests. This public repository includes a SHA-256 checksum manifest ([`SHA256SUMS.txt`](SHA256SUMS.txt)) for file-integrity verification of the published documents and sanitized figures (as an integrity check, not third-party authentication).
- **Internal Assurance Review:**  
  The package underwent internal read-only engineering reviews and disclosure checks to verify that mathematical formulations, signal interpretations, and reporting boundaries match the underlying synthetic traces without unstated assumptions.

---

## 2. Evidence Boundaries & Proprietary Withholding

This public repository is an **Engineering Showcase and Casebook**, not an open-source software release. To protect VolMax Studio's proprietary verification methods:

### What Is Published
- The canonical multi-use industrial BESS conflict formulation (`10.0 MWh` assumed nominal usable capacity).
- Three synthetic reference scenario summaries and sanitized visual comparisons ([Figures 1–3](assets/)).
- Detailed case studies ([Cases A, B, and C](cases/)) breaking down command-layer vs. physical-layer assessments.
- Mathematical conflict geometry and practical FAT/SAT commissioning checklists ([`ENGINEERING_BRIEF.md`](ENGINEERING_BRIEF.md)).

### What Is Intentionally Withheld
- **Proprietary Verification Code:** The automated Python verification software that evaluates telemetry traces against priority contracts.
- **Exact Numeric Tolerance Parameters:** The specific thresholds applied for floating-point deadbands, step tracking, and step-wise energy-balance checks.
- **Synthetic Generators & Negative-Control Suites:** The synthetic scenario generators and adversarial stress-test suites used during internal verification.
- **Raw CSV/JSON Data Traces:** Raw time-series evidence files remain in VolMax Studio's private assurance vault.

---

## 3. Explicit Non-Claims

1. **Not Field Validated:** All data shown are synthetic reference traces. This demonstration does not claim execution of a live EMS controller or field testing on operational battery hardware, utility SCADA systems, or commercial power plants.
2. **Not External Certification:** Internal engineering reviews do not constitute third-party certification, industrial accreditation, or endorsement by standards bodies (e.g. IEEE, IEC, or UL).
3. **No Public Self-Reproduction:** Because underlying verification software and raw datasets are withheld, external replay cannot be performed directly from this public repository.

---

## 4. Controlled Demonstration Protocol

For industrial BESS asset owners, commissioning managers, battery EPCs, or technical advisors interested in evaluating this assurance approach for FAT/SAT or operational log reviews:

VolMax Studio can arrange a **controlled demonstration of verifier execution** on reference scenarios or agreed datasets. Specific demonstration scope, evidence outputs, and any level of technical access are defined by separate written agreement (including mutual confidentiality where appropriate) and do not entail disclosure of proprietary decision rules or implementation code by default.

To inquire about a controlled demonstration, contact:  
📧 **VolMax Studio** — [`volmax.core@gmail.com`](mailto:volmax.core@gmail.com)
