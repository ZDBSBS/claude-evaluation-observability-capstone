# Perturbation Log

For each system, one deliberate change to an input or configuration was made and compared with the unperturbed behavior.

---

### System 1 — validated, routed pipeline

- **Change I made (file + what I changed):** Created `01-policy-pipeline/perturbation-policy.txt` from `POL-2025-001.txt` and removed the complete `ENDORSEMENTS` section so that the source genuinely contained no endorsement information.
- **Command I ran:** `.venv/bin/policy-extractor extract "01-policy-pipeline/perturbation-policy.txt" --policy-id POL-2025-001-PERTURBED`
- **What I predicted:** The missing required information would be classified as `missing_source` and escalated immediately rather than retried or fabricated.
- **What actually happened (paste the key output line):** `"category": "missing_source", "detected_pattern": "endorsements_absent", "field": "endorsements", "kind": "escalation"` with reason `"Field 'endorsements' returned null — the source document does not contain this information. Retry is futile; escalate to human review."` The run made one API call.
- **How this differs from the unperturbed run:** The baseline extracted three endorsements successfully with `retry_count: 0`. After the endorsements section was removed, the system returned no invented endorsements and escalated the missing-source case immediately.

---

### System 2 — schema-enforced two-pass extraction

- **Change I made (file + what I changed):** Created `02-mortgage-extraction/perturbation-income.txt` from `income_sum_mismatch.txt` and changed `TOTAL MONTHLY EARNINGS` from `10,892.17` to `9,642.17`, matching the sum of the monthly line items.
- **Command I ran:** `.venv/bin/mortgage-extract "02-mortgage-extraction/perturbation-income.txt" --mode record`
- **What I predicted:** Because the new stated total matched the calculated total of `9642.17`, the consistency validator would report the extraction as consistent with no discrepancy.
- **What actually happened (paste the key output line):** `"stated_monthly_total": 9642.17` followed by `"consistent": true` and `"discrepancies": []`.
- **How this differs from the unperturbed run:** The original document produced `"calculated": 9642.17`, `"stated": 10892.17`, `"delta": -1250.0`, and `"consistent": false`. After changing the stated total to match the line-item sum, the discrepancy disappeared.

---

### System 3 — multi-source synthesis

- **Change I made (file + what I changed):** Changed the investigation configuration by adding `--simulate-timeout`, deliberately making the logistics source unavailable.
- **Command I ran:** `.venv/bin/supply-chain-investigate meridian --offline --simulate-timeout`
- **What I predicted:** The investigation would continue without the logistics source, explicitly mark logistics as unavailable, and move logistics-dependent information to `Incomplete` rather than aborting the run.
- **What actually happened (paste the key output line):** `Sources unavailable: logistics unavailable (timeout)` and `late_shipment_count [missing source: timeout reading logistics]`. The briefing still completed.
- **How this differs from the unperturbed run:** The normal run used logistics data, reported `late_shipment_count` as `11.0 shipments`, and placed `on_time_delivery_rate` in `Contested` because supplier audit reported `95.0 percent` while logistics reported `78.0 percent`. With logistics unavailable, `late_shipment_count` moved to `Incomplete`, and `on_time_delivery_rate` remained only as the `95.0 percent` supplier-audit claim.
