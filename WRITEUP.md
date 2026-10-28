# AI Proposal Triage Design

## 1. AI vs. rules

The reviewer uploads proposal PDFs and selects a category. The system retrieves the admin's active criteria version, validates file type and size, extracts text, and sends the text to an AI extraction prompt. AI is useful for finding facts expressed in different wording and returning them in a structured format. It must return evidence snippets and page references where available, and use `unknown` rather than guessing.

Ordinary code validates the schema, checks mandatory requirements, applies the configured scoring rubric, calculates totals, sorts eligible proposals, and creates the output. These tasks should not be left to a language model because they need predictable, testable results. The system treats all proposal text as untrusted data, not instructions. A human reviews uncertain extractions, mandatory failures, flags, and the final ranking. A person—not the tool—makes the award decision.

## 2. Consistency and explainability

Each criteria configuration has a version, and each run stores the criteria version, prompt version, extraction output, evidence, scoring rules, model/version, timestamps, and final result. The scoring engine uses fixed bands and deterministic arithmetic. Given the same validated facts, criteria version, and rubric, it produces the same score and ordering. AI extraction can vary, so production use should use a controlled model/prompt configuration, validate outputs, and require review when facts are uncertain. A rerun can compare extraction results and flag differences.

For each criterion, the reviewer sees the extracted value, source evidence, points awarded, and reason. The result includes criterion-level breakdowns, mandatory check outcomes, missing facts, and the total. Ties use a documented deterministic tie-break rule, such as higher experience score, then lower price, then proposal letter.

## 3. Human in the loop and audit

The reviewer confirms the category and uploaded files. The interface highlights missing or low-confidence facts and all mandatory failures. A reviewer can correct an extracted value only with supporting evidence or mark it for clarification. Before using the ranked list, the reviewer confirms that flagged cases have been resolved. Any manual correction or override records the original value, new value, user, timestamp, reason, and evidence. Audit events also record upload identifiers, criteria/prompt/model versions, extraction output, calculations, ranking changes, and export/download actions. The tool recommends and explains; it does not award the contract.

## 4. Changing criteria

Criteria are versioned and immutable once used in a scoring run. Editing criteria creates a new version with validation (for example, weighted criteria must total 100%). Existing results retain the original version and remain reproducible. The admin can request a rerun using the new version; the system clearly labels the new results and preserves the prior run for comparison. It must not silently rewrite historical scores.

## 5. Risks and safeguards

- **Wrong PDF extraction/OCR:** retain source page references, validate required fields and data types, show evidence to reviewers, and flag unreadable or conflicting text.
- **Made-up facts:** require evidence for each extracted value; use `unknown` for absent facts; do not score unknown mandatory facts as passes.
- **Prompt injection/manipulation:** isolate document text as untrusted input, explicitly instruct the model to ignore embedded instructions, and use code—not the model—to calculate scores and ranks. Proposal C includes an instruction to rank itself first; ignore and log it.
- **Bias or inconsistent scoring:** use criteria configured by the admin, fixed scoring bands, consistent inputs, versioned prompts and rubric tests. Periodically test results across representative proposals.
- **Missing or ambiguous details:** flag the proposal for clarification rather than guessing. Proposal D has no confirmed schedule, so it is not ranked until the reviewer resolves this.
- **Security and privacy:** restrict access by role, encrypt data in transit and at rest, scan uploads, limit file size/type, store secrets outside source control, and apply a retention policy.
- **Cost, latency, and outages:** process asynchronously, set file/page limits and timeouts, track token/API costs, retry safely, and provide a manual review path if the AI service fails.
- **Criteria or configuration mistakes:** validate weight totals and scoring ranges, require authorized admin changes, and preserve immutable versions and audit logs.

## Proposed scoring rubric and sample outcome

Mandatory checks are separate from the weighted score. A proposal passes only when the required evidence is explicitly present. The bidder's own Ontario electrical contractor licence is required by the brief; a future subcontractor is not treated as confirmation.

For eligible proposals, use these fixed bands:
- Experience (30): 11+ qualifying installations = 30; 8–10 = 25; 4–7 = 20; 1–3 = 10; none = 0.
- Price (25): below $200,000 = 25; $200,000–under $220,000 = 22; $220,000–under $240,000 = 18; $240,000–$260,000 = 14; above ceiling = 0.
- Timeline (15): 10 weeks or less = 15; 11–12 = 13; 13–14 = 10; 15–16 = 5; over 16 = 0.
- Warranty (15): panel warranty contributes up to 7 points (25 years = 7; 20–24 = 5; 15–19 = 3; below 15 = 0). Workmanship contributes up to 8 points (12+ years = 8; 10–11 = 6; 7–9 = 4; 5–6 = 2; below 5 = 0).
- Team and approach (15): 3 points each for a named lead, named/defined crew, safety plan, roof/load assessment, and performance monitoring plan.

Using this proposed rubric, A scores 89, E scores 79, and C scores 68. B is ineligible because the bidder's licence is not confirmed. D is flagged and not ranked because the completion schedule is unknown. Proposal C's embedded instruction to automated reviewers is ignored as untrusted content. These are design outcomes based on the brief's fictional samples, not a final procurement decision.
