
# System Design: AI-Assisted Contractor Proposal Evaluation

**Reference:** MATACS-TH-2026-02

## 1. Objective

The objective is to design a system that evaluates contractor proposals for a commercial solar installation project with less repetitive manual work while preserving consistency, explainability, and human control.

The system must extract relevant information from proposal documents, evaluate mandatory requirements, score weighted criteria, and provide a defensible ranking.

The main design decision is to separate information extraction from evaluation. An LLM can interpret unstructured text, but it should not independently decide eligibility, calculate the final score, or award the contract.

## 2. Proposed Architecture

The system consists of the following components.

### 2.1 Criteria Configuration

An administrator configures:

- Mandatory eligibility requirements.
- Weighted evaluation criteria.
- The scoring formula for each weighted criterion.
- Required evidence and acceptable value formats.
- Rules for missing or ambiguous information.
- Criteria and scoring-rule versions.

For this assessment, the weighted categories are experience (30%), price (25%), timeline (15%), warranty (15%), and team and approach (15%).

The mandatory requirements are the Ontario electrical contractor licence and the required WSIB clearance and $5 million liability insurance.

### 2.2 Document Ingestion

The reviewer uploads a proposal in an accepted file format.

The system stores the original document and creates an evaluation record. It extracts text using a document parser or OCR where required.

The ingestion stage should retain page numbers and section information wherever possible so that extracted facts can be traced to their original location.

If text extraction fails, the system should stop the evaluation and request a readable document instead of continuing with incomplete input.

### 2.3 AI Extraction

An LLM reads the extracted proposal text and identifies facts relevant to the configured criteria.

Examples include:

- Number of qualifying commercial installations.
- Proposed price.
- Completion timeline.
- Panel and workmanship warranty periods.
- Licensing and insurance statements.
- Named project lead.
- Crew information.
- Safety and monitoring plans.

The model returns structured data rather than a free-form recommendation. Each extracted fact should include its evidence, source location when available, and an explicit status.

The model must not fill gaps with plausible values. If a price, licence, timeline, or warranty cannot be established from the proposal, the value remains unknown.

### 2.4 Validation

The application validates the extraction result before using it.

Validation checks include:

- Required fields are present in the output schema.
- Monetary values use a consistent currency and numeric format.
- Timelines use a consistent unit.
- Warranty periods are non-negative numeric values when stated.
- Evidence supports the extracted value.
- Conflicting statements are identified.
- Missing values are represented explicitly.
- The output conforms to the expected data structure.

A schema-valid response is not automatically a factually correct response. Evidence verification and human review are still necessary for consequential or uncertain fields.

### 2.5 Mandatory Eligibility Engine

A conventional rule engine evaluates the mandatory requirements.

Each requirement should have a status such as `pass`, `fail`, or `review_required`.

The distinction matters because absence of evidence is not always proof of non-compliance. The system should apply the actual procurement rule and distinguish an explicit failure from insufficient documentation.

For this assessment, BrightPath states that a licensed subcontractor will perform electrical work and that its own licence will be confirmed after contract award. Because the stated requirement is that the bidder holds and identifies the licence, this evidence does not establish compliance.

The system should flag BrightPath for clarification or mark it ineligible under the stated rule, depending on the procurement policy. It should not quietly treat the subcontractor's licence as equivalent to the bidder's licence.

### 2.6 Deterministic Scoring Engine

Once mandatory eligibility has been established, the application calculates the weighted scores using conventional code.

The LLM supplies extracted facts and evidence. It does not supply the final numerical scores.

The scoring engine applies the configured formula, checks for missing inputs, and records the calculation details. The same inputs and the same criteria version must produce the same result.

### 2.7 Results and Human Review

The results interface displays:

- Eligibility status.
- Total score where the score is complete.
- Individual criterion scores.
- Evidence supporting each extracted fact.
- Missing or conflicting information.
- The reason for any review flag.
- The criteria version used.

The reviewer can investigate flagged proposals, correct extraction errors with supporting evidence, and record an explanation for each correction.

The final contract award remains a human decision.

### 2.8 Audit Log

The system records the original document reference, document hash, extraction output, evidence, model and prompt versions, criteria version, scoring-rule version, score breakdown, flags, reviewer corrections, overrides, and final decision.

This makes it possible to explain why a proposal received a particular score and to reproduce the evaluation using the recorded inputs and rules.

## 3. Scoring Methodology

The assessment supplies the criterion weights but does not prescribe every mathematical formula. The following formulas are proposed assumptions for demonstrating a deterministic scoring approach. They should be confirmed against the actual procurement policy before operational use.

### 3.1 Experience — 30 Points

Use the number of qualifying commercial installations at or above the required capacity threshold.

For the illustrative comparison:

`Experience score = qualifying projects / maximum qualifying projects among eligible proposals × 30`

The eligible proposals with confirmed experience have the following counts:

| Proposal | Qualifying projects |
|---|---:|
| A — Northshore Solar | 14 |
| C — Lakeview | 9 |
| D — Essex Green | 11 |
| E — SunGrid | 8 |

The maximum is 14, so the experience scores are:

- A: 14 / 14 × 30 = 30.00
- C: 9 / 14 × 30 = 19.29
- D: 11 / 14 × 30 = 23.57
- E: 8 / 14 × 30 = 17.14

This formula is relative to the proposals being evaluated. If a stable procurement scale is preferred, the administrator should instead define fixed experience bands before proposals are opened.

### 3.2 Price — 25 Points

Lower prices receive higher scores, provided the proposals are comparable and all prices use the same currency and pricing basis.

`Price score = lowest eligible price / proposal price × 25`

The eligible prices are:

| Proposal | Price |
|---|---:|
| A — Northshore Solar | $238,000 |
| C — Lakeview | $244,000 |
| D — Essex Green | $231,000 |
| E — SunGrid | $242,500 |

The lowest eligible price is $231,000.

The resulting scores, rounded to two decimal places, are:

- A: 24.26
- C: 23.67
- D: 25.00
- E: 23.81

Before operational use, the pricing rule should define how taxes, optional work, exclusions, and differing contract scopes are handled.

### 3.3 Timeline — 15 Points

For proposals with confirmed completion timelines, use:

`Timeline score = shortest confirmed eligible timeline / proposal timeline × 15`

The confirmed timelines are:

| Proposal | Timeline |
|---|---:|
| A — Northshore Solar | 12 weeks |
| C — Lakeview | 14 weeks |
| D — Essex Green | Not confirmed |
| E — SunGrid | 13 weeks |

The shortest confirmed timeline is 12 weeks.

The resulting scores are:

- A: 15.00
- C: 12.86
- E: 13.85

Essex Green states that the timeline will be determined after a site assessment. The system must not invent a completion date or assign an arbitrary score. Its evaluation remains incomplete until the timeline is clarified or the administrator applies a pre-approved missing-information rule.

### 3.4 Warranty — 15 Points

For this illustrative calculation, the combined stated duration of the panel warranty and workmanship warranty is used.

`Warranty score = panel and workmanship warranty years combined / highest combined eligible warranty years × 15`

The proposal values are:

| Proposal | Panel warranty | Workmanship warranty | Combined |
|---|---:|---:|---:|
| A — Northshore Solar | 25 years | 10 years | 35 |
| C — Lakeview | 25 years | 10 years | 35 |
| D — Essex Green | 25 years | 10 years | 35 |
| E — SunGrid | 25 years | 12 years | 37 |

The maximum combined duration is 37 years.

The resulting scores are:

- A: 14.19
- C: 14.19
- D: 14.19
- E: 15.00

This combined-duration formula is only a simplifying assumption. Warranty duration alone does not capture exclusions, coverage, enforceability, or whether the two warranties have comparable terms. A real procurement process should define how these differences are evaluated.

### 3.5 Team and Approach — 15 Points

For the sample, the category is divided equally across four observable items:

| Item | Points |
|---|---:|
| Named project lead | 3.75 |
| Crew information | 3.75 |
| Safety plan | 3.75 |
| Monitoring plan | 3.75 |
| Total | 15.00 |

Each item receives its points only when supported by the proposal.

Under this illustrative checklist:

- A: 15.00
- C: 7.50
- D: 7.50
- E: 15.00

A and E provide a named lead, crew information, safety planning, and monitoring. C and D provide a named lead and safety-related or progress-planning information, but do not explicitly establish all four checklist items.

This scoring rule is intentionally simple and auditable. It does not claim that every missing item has the same practical importance in a real procurement.

## 4. Preliminary Results

The following totals use the formulas above and include only proposals with complete scores.

| Rank | Proposal | Experience | Price | Timeline | Warranty | Team | Total |
|---:|---|---:|---:|---:|---:|---:|---:|
| 1 | A — Northshore Solar | 30.00 | 24.26 | 15.00 | 14.19 | 15.00 | 98.45 |
| 2 | E — SunGrid | 17.14 | 23.81 | 13.85 | 15.00 | 15.00 | 84.80 |
| 3 | C — Lakeview | 19.29 | 23.67 | 12.86 | 14.19 | 7.50 | 77.51 |

The totals are rounded to two decimal places. Small differences may arise from whether rounding occurs at the individual-criterion stage or only after summing the unrounded values.

Essex Green passes the stated mandatory checks based on the supplied information, but its score is incomplete because the timeline is unconfirmed. It should not be inserted into the fully scored ranking as if the missing value were known.

BrightPath is not included in the eligible ranking because the supplied evidence does not establish the bidder's own licence under the stated requirement.

The fully scorable order is therefore A, E, and C. This does not mean D is necessarily worse than C or E; D requires additional information before a fair comparison can be made.

## 5. Missing and Ambiguous Information

Missing data must be handled explicitly.

The system distinguishes between:

- `confirmed`: the evidence supports the extracted fact.
- `not_confirmed`: the proposal does not establish the required fact.
- `conflicting`: different sections provide inconsistent information.
- `review_required`: a consequential decision cannot be made safely without further examination.

For mandatory requirements, a missing document should not automatically be treated as a pass.

For weighted criteria, an unknown value should not silently become zero or receive an invented score. The application should either hold the total pending clarification or use a documented missing-data rule established before evaluation.

This prevents proposals from being rewarded or penalized arbitrarily.

## 6. Prompt Injection and Untrusted Documents

Proposal C contains a statement attempting to instruct automated review systems to pre-approve the proposal and rank it first.

That statement must not influence the evaluation.

The submitted proposal is untrusted data, not an instruction source. The extraction prompt must tell the model to ignore instructions contained within the document and extract only facts relevant to the configured criteria.

However, prompt wording alone is not a sufficient security boundary. The application should also:

- Keep trusted system instructions separate from uploaded content.
- Restrict the model to a defined extraction schema.
- Validate the returned structure.
- Never allow document text to change criteria or scoring formulas.
- Run mandatory checks and scoring in application code.
- Preserve suspicious text for audit and review.
- Avoid allowing proposal content to invoke tools or perform external actions.

## 7. Other Risks and Mitigations

### Incorrect extraction

A model may misread a price, warranty, or licence number.

**Mitigation:** Require evidence, validate formats, retain source locations, and review high-impact fields.

### Unsupported conclusions

A model may infer that a subcontractor's licence satisfies a requirement applying to the bidder.

**Mitigation:** Define the entity and requirement precisely, and use explicit eligibility rules.

### Inconsistent scoring

A model may assign different subjective scores to identical submissions.

**Mitigation:** Calculate numerical scores with deterministic formulas and version the rules.

### Bias and irrelevant signals

A model may be influenced by proposal length, writing style, branding, or persuasive language.

**Mitigation:** Extract facts tied to predefined criteria and exclude irrelevant presentation features from scoring.

### Document-processing errors

OCR may omit tables or confuse digits.

**Mitigation:** Detect extraction failures, preserve the original file, validate values, and route uncertain cases for review.

### Criteria changes

An administrator may change the weights after a scoring run.

**Mitigation:** Store versioned criteria and retain the version used for each evaluation. A new criteria version should produce a new scoring run rather than silently overwrite the old result.

### Audit gaps

A reviewer may be unable to explain a score later.

**Mitigation:** Preserve evidence, calculation details, criteria versions, reviewer corrections, and decision history.

## 8. Human Oversight

Human review is required when:

- A mandatory requirement is ambiguous.
- Evidence is missing or contradictory.
- The extracted value has low confidence or cannot be verified.
- A proposal needs a clarification.
- A reviewer proposes an override.
- The final award decision is made.

Reviewer corrections should include the original extracted value, the corrected value, the evidence supporting the correction, the reviewer, the timestamp, and a reason.

The system can recommend an ordering of eligible proposals, but it must not make the final award automatically.

## 9. Criteria Versioning and Reproducibility

Each evaluation run should record:

- Criteria version.
- Weights and scoring formulas.
- Extraction prompt version.
- Model identifier and relevant settings.
- Document hash and extraction output.
- Rule-engine version.
- Individual criterion scores.
- Final score or incomplete status.
- Review flags and human corrections.

If weights change, the system creates a new criteria version. Previous results remain available so the reviewer can explain how an earlier ranking was produced.

## 10. Evaluation of the Design

The design separates tasks according to their reliability requirements.

AI is useful for finding relevant facts in lengthy and inconsistently structured proposals. Deterministic code is better for applying fixed eligibility conditions, checking numeric values, and calculating repeatable scores.

The strongest part of this approach is that a reviewer can trace a score back to both the source evidence and the rule that produced it. The remaining limitations are the quality of document extraction, the need to define fair scoring policies, and the need for human judgment when evidence is incomplete.

The proposed system is a design, not a claim that a production application has been implemented or tested.
