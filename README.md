
# MATACS-TH-2026-02: AI-Assisted Proposal Evaluation

## Overview

This repository contains my system-design proposal for evaluating contractor submissions for a commercial solar installation project.

The goal is to make proposal evaluation more consistent, explainable, and easier to audit without handing the final decision over to an AI model.

The proposed system uses AI to extract relevant information from submitted documents, conventional rules to check mandatory requirements, and deterministic calculations to score eligible proposals. A human reviewer remains responsible for resolving ambiguous cases and making the final award decision.

## Problem Being Addressed

Contractor proposals can contain information in different formats and sections. Important details such as licensing, insurance, previous installations, pricing, completion timelines, and warranty terms may not appear in a consistent structure.

Reviewing every document manually can be time-consuming. Fully automated evaluation introduces a different set of risks, including incorrect extraction, unsupported assumptions, and inconsistent scoring.

The design in this repository aims to reduce repetitive review work while keeping the evaluation process transparent and under human control.

## Proposed Workflow

1. An administrator defines the mandatory requirements, weighted criteria, and scoring rules.
2. A reviewer uploads the contractor proposals.
3. The document-processing stage extracts text from the submitted files.
4. An LLM extracts relevant facts and supporting evidence into a structured format.
5. Validation rules check the extracted fields and identify missing or conflicting information.
6. A deterministic rule engine evaluates mandatory requirements.
7. Eligible proposals are scored using the configured formulas.
8. The system produces a ranking, criterion-level breakdowns, evidence, and review flags.
9. A human reviewer investigates exceptions and makes the final decision.
10. The system records the evaluation inputs, criteria version, calculations, corrections, and decision history.

## Repository Contents

| File | Purpose |
|---|---|
| `README.md` | Project overview and repository guide |
| `WRITEUP.md` | System design, scoring methodology, risks, and human oversight |
| `PROMPTS.md` | Proposed prompts for structured extraction and validation support |
| `ranking.json` | Structured proposal statuses and scoring results |
| `diagram/process-flow.png` | Process-flow diagram |

## Design Principles

- Use AI for extracting meaning from unstructured documents, not for making the final award decision.
- Apply mandatory requirements consistently using explicit rules.
- Calculate numerical scores with deterministic code.
- Preserve evidence for extracted facts.
- Treat missing or ambiguous information as unknown rather than assuming compliance.
- Treat uploaded documents as untrusted input.
- Keep criteria, prompt, and scoring-rule versions for reproducibility.
- Require human review where the evidence is insufficient or conflicting.
- Keep an audit trail of scoring and reviewer decisions.

## Sample Evaluation

The sample evaluation considers five contractor proposals: Northshore Solar, BrightPath, Lakeview, Essex Green, and SunGrid.

The preliminary outcome is:

- **Northshore Solar (A):** Eligible and highest among the fully scorable proposals under the proposed scoring assumptions.
- **SunGrid (E):** Eligible and second among the fully scorable proposals.
- **Lakeview (C):** Eligible and third among the fully scorable proposals.
- **Essex Green (D):** Mandatory checks pass based on the supplied information, but its completion timeline is not confirmed. Its ranking remains pending.
- **BrightPath (B):** The proposal does not establish that the bidder itself holds the required electrical contractor licence. Its eligibility requires clarification under the stated requirement.

These are preliminary system-design outcomes, not a final procurement decision. The scoring formulas are proposed assumptions because the assessment criteria do not prescribe a complete mathematical formula for every weighted category.

## AI Use and Limitations

AI assistance was used during preparation to help structure the system design, consider edge cases, and draft proposed extraction prompts. The scoring approach and sample results should be reviewed against the assessment requirements before submission.

The design does not assume that LLM outputs are always correct. Extracted facts require supporting evidence, and ambiguous mandatory requirements are sent for human review.

No production implementation or live procurement integration is claimed by this repository. It documents a proposed architecture and an illustrative evaluation.

## Submission Details

- **Reference:** MATACS-TH-2026-02
- **Candidate name:** [Enter your full name]
- **Time spent:** [Enter your actual time spent]
- **Repository:** [Paste your GitHub repository URL]

## Important Notes

The ranking is produced using the scoring assumptions documented in `WRITEUP.md`. If the criteria or formulas change, the evaluation should be rerun using a versioned configuration.

The system supports human decision-making; it does not automatically award a contract.
