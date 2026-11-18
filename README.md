# Claude Evaluation and Observability Capstone

Evidence-based evaluation and observability capstone for reliable Claude-powered extraction and multi-source synthesis systems.

## Project Overview

This repository contains the evidence pack for three Claude-based reference systems:

1. **Validated Insurance Policy Extraction Pipeline**
   - Validation-driven retry and escalation
   - Independent review
   - Deterministic human-in-the-loop routing
   - Stratified sampling and calibration

2. **Resilient Mortgage Document Extraction**
   - Schema-enforced structured extraction
   - Two-pass classify-then-extract processing
   - Nullable fields for missing information
   - Normalization
   - Arithmetic-consistency validation

3. **Supply-Chain Risk Investigation**
   - Multi-source synthesis
   - Provenance preservation
   - Conflict annotation
   - Source-failure handling
   - Graceful degradation

## Verified Results

| System | Test Result | Static Analysis |
|---|---|---|
| Policy Pipeline | 45 passed, 3 live tests skipped | mypy and ruff passed |
| Mortgage Extraction | 25 passed | mypy and ruff passed |
| Supply-Chain Investigation | 34 passed | mypy and ruff passed |

## Key Evidence

### Policy Pipeline

The end-to-end run produced 9 routing decisions:

- 0 auto-approve
- 8 human-review
- 1 spot-check
- 1 escalation

The calibration report exposed an `umbrella / exclusions` slice with confidence `0.93` but observed accuracy `0.00`.

### Mortgage Extraction

The two-pass pipeline classified the appraisal document as `appraisal`.

Verified behavior includes:

- `about 2,400 sq ft` normalized to `2400`
- missing bonus returned as `null`
- calculated monthly income: `9642.17`
- stated monthly income: `10892.17`
- detected delta: `-1250.0`

### Supply-Chain Investigation

The generated briefing contains:

- Well-Established
- Contested
- Incomplete

A preserved conflict reports:

- `95.0 percent` from `supplier_audit`
- `78.0 percent` from `logistics`

The timeout experiment completed successfully while explicitly marking the logistics source as unavailable.

## Repository Structure

```text
.
├── README.md
├── environment.txt
├── perturbation-log.md
├── reflection-brief.md
├── 01-policy-pipeline/
├── 02-mortgage-extraction/
└── 03-supply-chain/
```