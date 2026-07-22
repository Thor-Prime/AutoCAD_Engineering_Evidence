# AI Context — AutoCAD Engineering Evidence

## Purpose

This repository preserves historical engineering evidence for the AutoCAD program. It contains source conversations, reviews, notes, investigations, and other records that may explain product intent, business rules, architectural decisions, abandoned approaches, and unresolved questions.

## Governing Distinction

- Historical evidence records what was said, attempted, observed, or believed at a point in time.
- Historical evidence is not automatically current truth.
- Current accepted engineering knowledge belongs in `Thor-Prime/AutoCAD_Governed_Solution`.
- Production behavior is established by the source code and verified runtime evidence.

## Required Review Order

Before mining evidence, read:

1. `README.md`
2. `AI_CONTEXT.md`
3. `Governance/EM-0001_Evidence_Mining_Methodology.md`
4. `Governance/Evidence_Header_Template.md`
5. `Governance/Evidence_Mining_Log.md`
6. The target source artifact
7. Any existing mining report for that artifact

## Evidence Handling Rules

1. Preserve original evidence content without rewriting it.
2. Store metadata and mining interpretation separately when modifying the original would violate immutability.
3. Distinguish:
   - Directly Supported Statement
   - Confirmed Human Intent
   - Inference
   - Hypothesis
   - Contradiction
   - Superseded Implementation
4. Do not promote code snippets as governing architecture merely because they appeared in a historical conversation.
5. Do not silently reconcile contradictions. Record them and request review.
6. Preserve provenance: source, date, participants, title, and original location when available.
7. Never commit confidential production data without explicit sanitization and approval.

## Mining Outputs

A mining report may recover:

- Business Rules
- Product Intent
- Product Requirements
- Architectural Decisions
- Architectural DNA
- Risks and Failure Modes
- Future Vision
- Dead Ends or Superseded Approaches
- Open Questions
- Promotion Candidates

## Promotion Rule

Nothing moves directly from historical evidence into production code. Promotion follows:

```text
Evidence
→ Mining
→ Human Review
→ Governed Documentation
→ Authorized Implementation
→ Verification
```

## Resume Instructions

A future reviewer shall:

1. Find the next `Not Reviewed` or `Mining In Progress` item in the mining log.
2. Read the source artifact completely enough to support the intended scope.
3. Update or create the mining report.
4. Record precise confidence and unresolved questions.
5. Update the mining log.
6. Propose promotion into the governed solution repository, but do not perform that promotion without authorization.
