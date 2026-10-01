# Reflection Brief — Evaluation and Observability Capstone

**Name:** Humaira
**Date:** September 2026

> Ground every answer in your own run. When a question asks for a number, file name, or line, paste it from your artifacts — a reviewer should be able to find it. Answers that are correct in the abstract but cite nothing do not meet the bar. Keep it short and specific.

---

## 0. Environment

| Field                        | Value                                                                                                                                |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| OS & version                 | Linux (Vocareum workspace)                                                                                                           |
| Python version               | Python 3.13                                                                                                                          |
| Date run                     | September 2026                                                                                                                       |
| Ran any system live? (which) | System 1 live pipeline attempted; stopped at Anthropic authentication. Systems 2 and 3 were run successfully in replay/offline mode. |

---

## 1. Validated, routed pipeline

| Evidence                                        | Value                                                                                                     |
| ----------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Passing test count                              | 45 passed, 3 skipped                                                                                      |
| Routing output file                             | `routing_decisions.json` was not generated because the live pipeline stopped at Anthropic authentication. |
| auto_approve / human_review / spot_check counts | Not available because no live routing output was generated.                                               |

**1a. Retry boundary.** From your perturbation run (a required field removed), paste the escalation record. How many API calls did the system make, and why is retrying a futile case worse than escalating it?

> Test: `test_ac_01_04_missing_source_halts_immediately`. The missing `endorsements` field produced `RetryFutileEscalation`, with field `endorsements` and detected pattern `endorsements_absent`. `client.call_count == 1`, so the system made exactly one API call. Retrying is futile because the required source field is absent; another identical request cannot create information that is not present. Escalation preserves the case for human handling instead of wasting calls.

**1b. Reading the router.** Pick one `human_review` record from your routing output. Which of the three signals (confidence, reviewer, integration) sent it to a human? If you had trusted the model's confidence alone, what would have happened?

> No live routing record was generated because the pipeline stopped at Anthropic authentication. The routing tests use `POL-B` as the low-confidence example: `premium_amount` has confidence `0.50`, below the `0.90` threshold, so the case is routed to `human_review`. The tests also verify that reviewer disagreement and integration failure independently cause human review. If confidence alone were trusted, a case with reviewer disagreement or integration failure could be incorrectly auto-approved.

**1c. Where the aggregate lies.** Run the calibration snippet. Quote the one cell whose accuracy lags its confidence, plus the overall figure. What does slicing by `policy_type × field` catch that a single number hides?

> Calibration output: `umbrella exclusions n=2 conf=0.93 acc=0.00 brier=0.865`; `OVERALL brier=0.291`. The slice exposes a field-specific problem hidden by the overall aggregate: the umbrella `exclusions` predictions were highly confident but had zero accuracy, while other fields performed differently.

---

## 2. Schema-enforced two-pass extraction

| Evidence           | Value                         |
| ------------------ | ----------------------------- |
| Passing test count | 25 passed                     |
| Document run       | `appraisal_informal_sqft.txt` |
| Classified type    | `single_family`               |

**2a. Two guarantees.** Paste your discrepancy-run output. Tool use already forces valid JSON, yet the validator still catches a bad sum. Why are these two different guarantees? Name one error each cannot catch.

> `income_sum_mismatch.txt`: base `5416.67`, bonus `1250`, commission `2140`, overtime `385.5`, other `450`; stated total `10892.17`; calculated total `9642.17`; delta `-1250.0`; `consistent: false`. Valid JSON guarantees syntactic/schema structure, but it does not guarantee that related numeric fields are mathematically consistent. Conversely, the consistency validator can catch a bad sum but cannot catch every structural or type error that JSON/schema validation catches.

**2b. Refusing to fabricate.** Run on a document missing a field. Paste that field's output. Why null instead of an invented value? Point to the schema choice that allows it.

> `income_missing_bonus.txt` produced `bonus_monthly: null` and `bonus_ytd: null`; `commission_monthly: null`; `overtime_monthly: null`; and `other_monthly: null`. The schema allows missing values to be represented as nullable/optional fields, so the extractor can preserve uncertainty instead of inventing a value.

**2c. Normalization.** Quote one field where the source text and extracted value differ in format ("about 2,400 sq ft" → `2400`). Why normalize at extraction time rather than downstream?

> `gross_living_area_sqft`: source text was “about 2,400 sq ft” and the extracted value was `2400`. Normalizing during extraction gives downstream validation and consumers a consistent numeric representation instead of requiring every later step to interpret units, punctuation, and approximate wording independently.

---

## 3. Multi-source synthesis

| Evidence                       | Value                                              |
| ------------------------------ | -------------------------------------------------- |
| Passing test count             | 34 passed                                          |
| Briefing file                  | `capstone-submission/03-supply-chain/briefing.txt` |
| Section the conflict landed in | `## Contested`                                     |

**3a. Annotate, don't arbitrate.** Quote one conflicting-metric pair from your briefing — both values, sources, dates. Give one way a reader is better served by the preserved conflict than by a single reconciled number.

> `on_time_delivery_rate`: `95.0 percent` from `supplier_audit` dated `2026-04-10` versus `78.0 percent` from `logistics` dated `2026-04-05`. Preserving both values and their provenance lets the reader see that the disagreement is real and decide what follow-up is required instead of hiding the discrepancy behind an unexplained reconciled number.

**3b. Source goes dark.** Run with `--simulate-timeout`. Paste the part of the briefing showing the failed source. How is "unreachable" handled differently from "nothing to report," and why does the run still finish?

> Timeout output: `Sources unavailable: logistics unavailable (timeout)` and `late_shipment_count [missing source: timeout reading logistics]`. An unreachable source is explicitly annotated as unavailable rather than interpreted as evidence that there were no late shipments. The run still finishes because the coordinator retains surviving source results and marks only the dependent information as incomplete.

**3c. Dates as a guardrail.** Quote two claims about the same supplier with different dates. How does requiring a date stop a time difference from reading as a contradiction?

> `on_time_delivery_rate` was `95.0 percent` in the supplier audit dated `2026-04-10`, while the logistics source reported `78.0 percent` dated `2026-04-05`. The dates show that the measurements were not necessarily made at the same time, so a difference can represent a change over time rather than an immediate contradiction.

---

## 4. Synthesis

**4a. One principle.** Name the single moment in your runs (system + artifact) where *evaluate the output, don't trust the model's word* most clearly caught something a trusting design would have shipped.

> System 2, `income_sum_mismatch.txt`, caught a mathematical inconsistency that valid JSON alone would not catch: the stated monthly income was `10892.17`, while the calculated total was `9642.17`, with delta `-1250.0` and `consistent: false`. A trusting design could have accepted the syntactically valid extraction.

**4b. Confidence ≠ correctness.** Pick the system where this mattered most, and explain why using something you observed.

> System 1. The calibration report showed `umbrella exclusions n=2 conf=0.93 acc=0.00 brier=0.865`, despite the high average confidence of `0.93`. The overall Brier score was `0.291`. This demonstrated directly that a high confidence value can coexist with incorrect predictions, which is why confidence needs calibration and evaluation.

**4c. Apply it.** Describe a real workflow where an LLM pulls structured results from messy input. Which pattern — validated retry with escalation, independent review with deterministic routing, or provenance-preserving conflict annotation — would you reach for first, and what would you instrument to know when it broke?

> A useful workflow is extracting structured insurance, mortgage, or supplier-risk information from messy PDFs, emails, and forms. I would use independent review with deterministic routing when the extracted information drives a business decision: high-confidence results can proceed only when validation passes and there is no reviewer disagreement or integration failure; otherwise the case goes to human review. I would instrument field-level confidence, reviewer disagreements, validation failures, integration failures, escalation counts, retry counts, and calibration metrics such as accuracy and Brier score so that failures and overconfidence are visible.
