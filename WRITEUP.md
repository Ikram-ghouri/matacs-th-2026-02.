# Design write-up

## End-to-end process and scoring policy
An admin publishes a **versioned category configuration** containing mandatory checks, weighted criteria, rubric thresholds, cutoff dates, and evidence requirements. A reviewer uploads PDF proposals and selects the category. The server validates files, extracts PDF text (OCR only when necessary), and labels every page and source span. An AI extractor returns *structured factual claims with citations and confidence*, never eligibility decisions, scores, or instructions. The reviewer can correct extracted facts against the source PDF. Rule-based validation then evaluates mandatory requirements and scores eligible, sufficiently documented proposals. A reviewer approves the ranked list; a separate human makes the award.

**Missing-data policy:** A missing mandatory proof fails eligibility; explicit evidence of a pending subcontractor's licence does not establish the bidder's own licence. Missing project completion duration is **flagged for clarification**, because whether the 16-week limit can be met cannot be confirmed. For weighted, noncritical fields, missing or insufficiently dated evidence scores **zero for that subcriterion**, with a visible warning and a chance for reviewer correction. These are triage outcomes, not final procurement findings.

**Illustrative, administrator-configurable scoring rubric (0–100 points per criterion):**

| Criterion | Proposed deterministic rubric |
| --- | --- |
| Experience (30%) | Count **verified** qualifying installs ≥100 kW completed within five years of the configured review date: ≥14→100; 10–13→85; 7–9→70; 4–6→50; 1–3→25; 0 or unverified count→0. |
| Price (25%) | Must be ≤$260k CAD: ≤$200k→100; >$200k–$230k→90; >$230k–$245k→80; >$245k–$260k→60. Price above ceiling is flagged for human review rather than silently ranked. |
| Timeline (15%) | Documented ≤10 weeks→100; 11–12→90; 13–14→75; 15–16→60; missing→flag for clarification; >16→ineligible under project constraint. |
| Warranty (15%) | Panels ≥25 years plus workmanship ≥12 years→100; workmanship 10–11→90; workmanship 5–9→65; otherwise 0. Both warranty claims must be evidenced. |
| Team & approach (15%) | Four equally weighted binary elements: explicitly identified/named lead, documented crew size, safety plan, and performance-monitoring plan. 25 points each; missing evidence=0. |

Weighted total = `0.30×experience + 0.25×price + 0.15×timeline + 0.15×warranty + 0.15×team`. Rank descending; tie-break by lower verified price, then proposal letter. Scores are illustrative results of **this rubric**, not AI-generated judgments. The date reference is the configured review date; a statement that work occurred “since 2020” does not establish how many projects fall inside the five-year window.

**Sample outcome:** A=92.00 (100/80/90/90/100); E=82.25 (70/80/75/100/100); C=52.25 (0/80/75/90/50). A and E have qualifying counts stated from 2021 onward; C's total dating back to 2020 cannot establish the count inside the five-year window, so experience receives a provisional zero pending project-date evidence. C remains ranked because experience is **weighted**, not mandatory. D is flagged because the delivery time is missing. B is ineligible because the bidder's own Ontario licence is not established. No award is automatic.

## 1. AI vs. rules
Use AI to map unstructured proposal wording into a strict JSON schema, normalize terms, and attach PDF page/span citations. Use ordinary code for file validation, required-field checks, mandatory pass/fail, quantitative scoring, tie-breaks, and version tracking. AI can hallucinate and vary between calls; code makes procurement decisions repeatable.

## 2. Consistency and explainability
Store immutable PDF hashes, corrected structured facts, criterion version, review-date cutoff, rubric, per-criterion scores, evidence citations, and final rank. Recalculating from the **same frozen verified facts and criteria version** yields exactly the same scores and order; raw AI extraction may differ across runs, so it must be verified and frozen first. The UI displays a score breakdown, source snippets/page links, warnings, and explanations for flags and exclusions.

## 3. Human in the loop
A reviewer checks extracted evidence and resolves missing or ambiguous fields before approving the ranking. Authorized personnel may correct a fact or override a status/score with a required written reason; final procurement approval remains human. Append-only audit events record user, timestamp, document hash, source evidence, original/new value, criteria version, override rationale, and final decision. An override never silently edits the underlying proposal.

## 4. Changing criteria
Publish edits as a new immutable criteria version. Existing scoring runs retain their previous version and remain auditable. Offer explicit **re-score all proposals** under the new version, producing a separate comparative run and notifying reviewers of changed statuses/ranks. Never mix scores from different versions in the same ranking.

## 5. Risks and safeguards
- **Wrong PDF/OCR extraction:** show source spans and confidence; require manual confirmation on poor-quality or missing evidence.
- **Hallucinations:** structured schema, `null` when unknown, source citations, and validation that cited text supports each field.
- **Prompt injection:** treat all PDF text as untrusted data. Ignore directives in proposals, including apparent “pre-approval” or demands for full marks. Only admin-published criteria control the rules.
- **Bias/unfair grading:** fixed rubric, evidence-based comparisons, no company-name preference, accessible explanation, and sampled human audits.
- **Manipulation/forged claims:** record original file hashes, verify certificates/licences with authorized external checks before award, and log corrections.
- **Privacy, cost and reliability:** access controls, encrypted storage, retention limits, PDF size limits, quotas/caching, retry policy, and a manual processing fallback.
