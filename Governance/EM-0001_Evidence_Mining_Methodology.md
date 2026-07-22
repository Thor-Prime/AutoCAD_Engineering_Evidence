# EM-0001 — Evidence Mining Methodology

## Status

Initial governed method for mining historical AutoCAD engineering evidence.

## Objective

Recover durable engineering knowledge from historical records without treating those records as automatically correct or current.

## Standard Workflow

1. **Register** — assign a permanent evidence ID and record provenance.
2. **Preserve** — retain the source artifact without rewriting its historical content.
3. **Classify** — identify source type, confidentiality, scope, and reliability.
4. **Mine** — extract only knowledge supported by the artifact.
5. **Separate epistemic status** — distinguish direct support, confirmed intent, inference, hypothesis, contradiction, and superseded implementation.
6. **Review** — present meaningful recovered knowledge and open questions to the repository owner.
7. **Promote** — after approval, transfer accepted knowledge into the governed solution repository.
8. **Trace** — update both the evidence record and governed artifact with reciprocal references.

## Standard Mining Categories

### Business Rule

A constraint or invariant the product must protect.

### Product Intent

The user outcome, engineering purpose, or operational experience the system is intended to provide.

### Product Requirement

A capability the product is expected to support.

### Architectural Decision Candidate

A design choice that may deserve a formal ADR after current-state verification.

### Architectural DNA

An enduring design principle that survives individual implementations.

### Risk or Failure Mode

A condition likely to cause corruption, duplicate authority, crashes, user confusion, or migration failure.

### Future Vision

A desired capability not yet demonstrated as implemented.

### Dead End or Superseded Approach

An implementation or idea that should not be repeated without new evidence.

### Open Question

A material uncertainty requiring code verification, artifact comparison, or human clarification.

## Confidence Scale

- **High:** Explicitly stated by the repository owner or repeatedly supported by concrete evidence.
- **Moderate:** Strongly supported but dependent on interpretation or incomplete context.
- **Low:** Plausible hypothesis requiring verification.

## Promotion Restrictions

A historical chat response may contain confident but incorrect recommendations. Therefore:

- Code proposals are implementation evidence, not accepted architecture.
- Assistant summaries are interpretations unless confirmed by the user or current repository evidence.
- User statements describing intended product behavior are primary evidence of human intent.
- Current code and runtime behavior must be verified separately.

## Artifact Layout

```text
ChatGPT/
└── AC-####_<title>/
    ├── SOURCE.md
    ├── METADATA.md
    └── MINING_REPORT.md
```

When the source file is too large or is uploaded separately, `METADATA.md` shall record the exact expected source filename and preservation status.

## Completion States

- Not Reviewed
- Mining In Progress
- Mined — Awaiting Human Review
- Reviewed — Promotion Pending
- Fully Mined
- Superseded Mining Report
