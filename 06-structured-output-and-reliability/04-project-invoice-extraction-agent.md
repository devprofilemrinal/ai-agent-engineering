# Phase 06 Project — Invoice Extraction Agent

## Scenario

Your finance team manually re-keys data from vendor invoices — PDFs and emailed text in wildly inconsistent formats — into your accounting system, and it's slow and error-prone. This project builds a Java service that extracts structured data from realistic, messy invoice text using everything in this phase: a schema designed for reliability, a three-stage validation pipeline, and a retry strategy that responds intelligently to each kind of failure. Unlike earlier projects in this handbook, this one is explicitly evaluated against a stress-test set built to break a naive implementation — the point is not to get a demo working once, but to characterize exactly where and how it fails, and to handle those failures deliberately rather than by accident.

## Functional requirements

1. **Design a schema following every principle from file 1**: flattened where reasonable, enums for fields with a fixed value space (e.g., `currency`, `payment_status`), explicit nullability for genuinely optional fields (e.g., `purchase_order_number`), and a properly structured `line_items` array where the one-to-many relationship is real.
2. **Implement the full three-stage validation pipeline from file 2**: syntactic (with prose/code-fence stripping), schema (with strict Jackson configuration that fails loudly on missing or null primitives), and semantic (at minimum: positive total amount, line items reconciling with the stated total within a small tolerance, and non-blank vendor name).
3. **Implement the corrective retry strategy from file 3**: a bounded retry loop where each retry's prompt names the specific validation failure from the previous attempt, distinguishing corrective phrasing by failure category (syntactic vs. schema vs. semantic) rather than using one generic "try again" message for every case.
4. **Build a human-review queue** for items that exhaust the retry budget without producing valid output — a simple persisted list is sufficient, but it must capture the source text, the final error message, and which validation stage it failed at, so a human reviewer has enough information to actually resolve it without re-deriving your pipeline's diagnosis from scratch.
5. **Construct a stress-test set of at least 12 realistic invoice text samples**, deliberately including: a normal, clean invoice; an invoice with a missing purchase order number; an invoice where line items don't sum to the stated total (a real typo case, not a hypothetical); an invoice in a currency your schema's enum doesn't include; an invoice with unusual formatting (line items in a table-like layout with inconsistent spacing); and at least one sample deliberately worded ambiguously enough that you'd expect a real extraction attempt to need at least one retry.
6. **Report results per stress-test sample**: which validation stage (if any) it failed at on the first attempt, how many retries it took to resolve (if it did), and whether it ultimately succeeded or landed in the human-review queue.

## Constraints

- Semantic validation must include the line-items-reconcile-with-total check specifically — this is the highest-value check in the entire pipeline and its absence would defeat the project's core purpose.
- The retry loop must have a hard-capped maximum attempt count, and every item that exceeds it must land in the human-review queue rather than being silently discarded, silently accepted with unresolved validation failures, or retried indefinitely.
- Corrective retry prompts must differ meaningfully by failure category — a single generic "there was an error, please try again" prompt used for every failure type does not satisfy this requirement; graders (or your own review) should be able to see the corrective prompt naming the specific field or discrepancy involved.

## What "done" looks like

- A results table across all 12+ stress-test samples showing, per sample, the first-attempt outcome, the number of retries used, and the final resolution (success or human-review queue).
- At least one stress-test sample that fails on the first attempt and is successfully corrected by a subsequent retry, with the retry's corrective prompt visibly referencing the specific issue detected — proving the corrective-retry mechanism actually functions, not just that it's implemented in theory.
- At least one stress-test sample that correctly lands in the human-review queue after exhausting retries, with a review-queue entry containing enough diagnostic detail (source text, failure stage, error message) that a human could act on it without re-running your pipeline's internals.
- The reconciliation check (line items vs. stated total) demonstrably catching the deliberately-inserted mismatched-total sample from your stress-test set.

## Extension

Extend the semantic validation layer with a cross-invoice consistency check: if you're processing invoices from the same vendor across multiple documents, flag any invoice whose vendor name is a near-miss variant of a previously seen vendor name (e.g., "Acme Corp" vs. "Acme Corporation" vs. "ACME CORP.") rather than silently treating them as distinct vendors — a realistic downstream data-quality problem that pure single-document validation, as built in this project, can't catch on its own, and a natural bridge toward the evaluation-at-scale thinking Phase 10 covers for entire agent systems rather than single extraction calls.