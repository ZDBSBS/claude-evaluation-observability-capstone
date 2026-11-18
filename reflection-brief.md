# Reflection Brief — Evaluation and Observability Capstone

**Name:** Zied Besbes
**Date:** 2026-11-18

> Ground every answer in your own run. When a question asks for a number, file name, or line, paste it from your artifacts — a reviewer should be able to find it. Answers that are correct in the abstract but cite nothing do not meet the bar. Keep it short and specific.

---

## 0. Environment

| Field | Value |
|---|---|
| OS & version | Linux 6.6.97+ x86_64 GNU/Linux |
| Python version | Python 3.13.0 |
| Date run | 2026-11-18 |
| Ran any system live? (which) | Yes — validated, routed policy pipeline; the mortgage perturbation also used record mode. Supply-chain investigation used offline mode. |

---

## 1. Validated, routed pipeline

| Evidence | Value |
|---|---|
| Passing test count | 45 passed, 3 skipped |
| Routing output file | `01-policy-pipeline/routing_decisions.json` |
| auto_approve / human_review / spot_check counts | 0 / 8 / 1 |

**1a. Retry boundary.** From your perturbation run (a required field removed), paste the escalation record. How many API calls did the system make, and why is retrying a futile case worse than escalating it?

> `01-policy-pipeline/perturbation-run.txt` contains `"category": "missing_source"`, `"detected_pattern": "endorsements_absent"`, `"field": "endorsements"`, and `"kind": "escalation"`. The reason states: `"Field 'endorsements' returned null — the source document does not contain this information. Retry is futile; escalate to human review."` The perturbation made one API call. A retry cannot recover information that is genuinely absent from the source, so additional calls would add cost and latency without providing evidence for a valid value.

**1b. Reading the router.** Pick one `human_review` record from your routing output. Which of the three signals (confidence, reviewer, integration) sent it to a human? If you had trusted the model's confidence alone, what would have happened?

> In `01-policy-pipeline/routing_decisions.json`, `POL-2025-002` was routed to `human_review`. Its `fields_below_threshold` and `integration_failures` lists were empty, but `reviewer_disagreements` contained `coverage_limit` and `deductible`. The reviewer signal therefore caused the routing decision. If confidence alone had been trusted, that independent disagreement would not have triggered human review.

**1c. Where the aggregate lies.** Run the calibration snippet. Quote the one cell whose accuracy lags its confidence, plus the overall figure. What does slicing by `policy_type × field` catch that a single number hides?

> `01-policy-pipeline/calibration-report.txt` shows `umbrella exclusions n=2 conf=0.93 acc=0.00 brier=0.865`, while the overall Brier score is `0.291`. The slice exposes a high-confidence subgroup that is consistently wrong even though the overall metric looks substantially better.

---

## 2. Schema-enforced two-pass extraction

| Evidence | Value |
|---|---|
| Passing test count | 25 passed |
| Document run | `02-mortgage-extraction/classification-run.txt` |
| Classified type | `appraisal` |

**2a. Two guarantees.** Paste your discrepancy-run output. Tool use already forces valid JSON, yet the validator still catches a bad sum. Why are these two different guarantees? Name one error each cannot catch.

> `02-mortgage-extraction/discrepancy-run.txt` reports `"field": "total_monthly_income"`, `"calculated": 9642.17`, `"stated": 10892.17`, `"delta": -1250.0`, with `"consistent": false`. Schema/tool enforcement guarantees the expected structure and types, but structurally valid JSON can still contain values that are mathematically inconsistent. Conversely, the arithmetic validator checks the relationship between extracted values but does not replace schema enforcement for malformed structure or incorrect field types.

**2b. Refusing to fabricate.** Run on a document missing a field. Paste that field's output. Why null instead of an invented value? Point to the schema choice that allows it.

> `02-mortgage-extraction/missing-field-run.txt` contains `"bonus_monthly": null`. The field is absent from the source, so returning `null` preserves the distinction between missing evidence and an asserted value. The extraction schema deliberately supports nullable fields, allowing the model to represent source silence without fabrication.

**2c. Normalization.** Quote one field where the source text and extracted value differ in format ("about 2,400 sq ft" → `2400`). Why normalize at extraction time rather than downstream?

> In `02-mortgage-extraction/classification-run.txt`, the input is classified as `appraisal`, extraction uses `extract_appraisal`, and `gross_living_area_sqft` is returned as `2400` from the informal source wording “about 2,400 sq ft”. Normalizing during extraction gives downstream validation and consumers one predictable structured representation rather than requiring each downstream component to interpret the same source formatting independently.

---

## 3. Multi-source synthesis

| Evidence | Value |
|---|---|
| Passing test count | 34 passed |
| Briefing file | `03-supply-chain/briefing.md` |
| Section the conflict landed in | `Contested` |

**3a. Annotate, don't arbitrate.** Quote one conflicting-metric pair from your briefing — both values, sources, dates. Give one way a reader is better served by the preserved conflict than by a single reconciled number.

> `03-supply-chain/briefing.md` reports `on_time_delivery_rate` as `95.0 percent — supplier_audit (as of 2026-04-10)` and `78.0 percent — logistics (as of 2026-04-05)`. Preserving both values and their provenance lets the reader see the disagreement directly and investigate why the sources differ instead of hiding uncertainty behind one reconciled number.

**3b. Source goes dark.** Run with `--simulate-timeout`. Paste the part of the briefing showing the failed source. How is "unreachable" handled differently from "nothing to report," and why does the run still finish?

> `03-supply-chain/timeout-run.txt` states `Sources unavailable: logistics unavailable (timeout)` and marks `late_shipment_count` as `[missing source: timeout reading logistics]`. By contrast, `production_capacity_utilization` is marked `[missing source: no source reported this metric]`. A timeout therefore records unavailable evidence rather than treating the source as evidence that nothing happened. The coordinator continues with the remaining sources, so the briefing can finish while preserving the missing-source limitation.

**3c. Dates as a guardrail.** Quote two claims about the same supplier with different dates. How does requiring a date stop a time difference from reading as a contradiction?

> For Meridian, supplier audit reports `on_time_delivery_rate = 95.0 percent` as of `2026-04-10`, while logistics reports `78.0 percent` as of `2026-04-05`. Retaining the dates makes the temporal context explicit. A reader can see that the observations were recorded on different dates instead of automatically assuming they describe exactly the same point in time.

---

## 4. Synthesis

**4a. One principle.** Name the single moment in your runs (system + artifact) where *evaluate the output, don't trust the model's word* most clearly caught something a trusting design would have shipped.

> The clearest example is the mortgage validator in `02-mortgage-extraction/discrepancy-run.txt`. The extraction was structurally valid, but the validator calculated `9642.17` against a stated total of `10892.17` and surfaced a `-1250.0` discrepancy. A design that accepted structured output without evaluating the values could have shipped that inconsistency.

**4b. Confidence ≠ correctness.** Pick the system where this mattered most, and explain why using something you observed.

> The policy pipeline showed this most clearly. In `01-policy-pipeline/calibration-report.txt`, the `umbrella / exclusions` slice had confidence `0.93` but observed accuracy `0.00` across two samples. High confidence therefore did not imply correctness, while slicing exposed the weakness that the overall Brier score of `0.291` could hide.

**4c. Apply it.** Describe a real workflow where an LLM pulls structured results from messy input. Which pattern — validated retry with escalation, independent review with deterministic routing, or provenance-preserving conflict annotation — would you reach for first, and what would you instrument to know when it broke?

> For a workflow that extracts structured fields from insurance renewal documents, I would start with validated retry with escalation. The policy perturbation showed why: when a required field was genuinely absent, the system identified `missing_source` and escalated after one API call instead of fabricating a value or retrying futilely. I would instrument validation-failure categories, retry counts, missing-source escalations, per-field confidence, human-review rates, and calibration slices so that changes in failure patterns become observable.
