# AC-0001 — Mining Report

## Status

Mined — Awaiting Human Review

## Scope

This report mines the historical conversation titled **StoreGuards Architecture Review**. The conversation began as a focused NOD-binding repair and expanded into a broad ProjectStore product and schema architecture discussion.

The report preserves a strict distinction between:

- **Confirmed Human Intent** — explicitly stated or accepted by Brian Ranstead;
- **Direct Observation** — demonstrable from code or artifacts included in the conversation;
- **Architectural Inference** — a strong interpretation requiring current repository verification;
- **Assistant Proposal** — historically suggested but not automatically accepted as current architecture.

## Executive Recovery

The thread reveals that ProjectStore was intended to become more than a tag counter or file path. It was evolving toward a **project-wide engineering authority** that:

- binds each drawing once to a project store;
- allows normal tools to resolve project context without repeated browsing;
- organizes plant objects by project-defined process systems;
- allocates and tracks line, valve, and equipment identities;
- records object placement and lifecycle data;
- models off-page connections across P&ID pages;
- uses Sheet Set data as the project drawing inventory;
- supports staged project bootstrap before the drawing inventory is complete.

This historical direction is strongly consistent with the current governed intelligent-P&ID product intent, but individual historical schemas and code proposals must not be adopted without reconciliation.

---

# 1. Recovered Business Rules

## BR-AC0001-001 — Bind once; resolve thereafter

**Status:** Confirmed Human Intent  
**Confidence:** High

Project setup or binding is responsible for writing the project-store location into the drawing. Normal operational tools should retrieve that binding from the drawing and should not repeatedly ask the user to browse for the file.

**Promotion candidate:** PDR-001 — Project Setup and Runtime Context

## BR-AC0001-002 — Normal tools do not prompt for the ProjectStore

**Status:** Confirmed Human Intent  
**Confidence:** High

Interactive file selection belongs to explicit setup, repair, attach, or administration workflows. Routine tools should resolve project context non-interactively and stop cleanly when the context is unavailable.

**Promotion candidate:** PDR-001; ADR candidate for interaction boundaries

## BR-AC0001-003 — One canonical ProjectStore path binding

**Status:** Architectural Inference supported by user-approved direction  
**Confidence:** High

Legacy NOD locations may be read for compatibility, but new writes must converge on one legal canonical binding. Competing path authorities create split-brain behavior.

**Current verification required:** Confirm current canonical key and all remaining writers in the governed solution.

## BR-AC0001-004 — Systems are project-defined process areas

**Status:** Confirmed Human Intent  
**Confidence:** High

A system represents a functional plant/process area such as dehydration, LNG storage, or ethane receipt. Project personnel configure systems for the specific project.

**Promotion candidate:** PDR-001 — Domain Vocabulary

## BR-AC0001-005 — Systems govern ranges for LINE, VALVE, and EQUIPMENT

**Status:** Confirmed Human Intent  
**Confidence:** High

The selected category name is `LINE`, not `PIPELINE`. Each system may contain allocation ranges for lines, valves, and equipment.

## BR-AC0001-006 — System range categories are flexible

**Status:** Confirmed Human Intent  
**Confidence:** High

A system may omit one or more categories. Missing range configuration means that category is not allocatable for that system; all categories are not universally required.

## BR-AC0001-007 — Starter systems exist to guide project setup

**Status:** Confirmed Human Intent  
**Confidence:** High

New projects should contain a small set of example/default systems that project users can use as models and edit for the project.

**Open design point:** Whether examples are active systems, disabled systems, or separate templates requires current confirmation. The assistant proposed `SystemTemplates`, but the user’s enduring requirement is the presence of editable examples.

## BR-AC0001-008 — Tags use one unified registry across object categories

**Status:** Confirmed Human Intent  
**Confidence:** High

The user selected one unified `Tags[]` collection with a category field rather than independent valve, line, and equipment arrays.

## BR-AC0001-009 — A tag record carries identity, provenance, placement, and lifecycle data

**Status:** Confirmed Product Intent; exact fields historically proposed  
**Confidence:** High for capability, Moderate for field contract

When a tag is assigned, ProjectStore is expected to preserve appropriate metadata, including the allocated identity, system/category, user/time provenance, drawing placement references, parent/child relationships where applicable, and lifecycle state.

**Caution:** The exact historical field list remains a schema proposal, not a frozen current contract.

## BR-AC0001-010 — An OPC ID represents one logical cross-page connection

**Status:** Confirmed Human Intent  
**Confidence:** High

An off-page connector ID is the shared identity that links two P&ID pages.

## BR-AC0001-011 — A valid OPC has two page-specific placements

**Status:** Confirmed Human Intent  
**Confidence:** High

One logical OPC normally has exactly two physical records/placements: one on each connected page. Both placements share the same OPC ID.

## BR-AC0001-012 — OPC numbers increment per page/use the project’s defined numbering policy

**Status:** Confirmed Human Intent, wording requires refinement  
**Confidence:** Moderate

The user described a numerical OPC number associated with the page workflow and a unique shared OPC ID. The precise allocation scope needs final normalization because the conversation uses both “increment per page” and “shared unique ID across two pages.”

## BR-AC0001-013 — OPC endpoint choices are governed selections, not free text

**Status:** Confirmed Human Intent  
**Confidence:** High

External endpoints and system endpoints should be selected from configured lists/dropdowns. External project boundaries are represented as controlled battery-limit endpoints.

## BR-AC0001-014 — Page number is globally unique in the active project Sheet Set

**Status:** Confirmed Human Intent  
**Confidence:** High

The user selected the option that sheet/page numbers are globally unique across the project Sheet Set. Duplicate page numbers are therefore a conflict rather than an ambiguity tools may resolve heuristically.

## BR-AC0001-015 — Preliminary projects may exist before Sheet Set synchronization

**Status:** Confirmed Human Intent  
**Confidence:** High

A drawing/project may be bound to a ProjectStore before a Sheet Set is attached or synchronized. This accommodates preliminary drawing development.

## BR-AC0001-016 — Full project-wide tools depend on drawing-inventory readiness

**Status:** Architectural Inference accepted through bootstrap-state discussion  
**Confidence:** High

Project-wide operations such as OPC navigation, audits, and batch processing should distinguish a bound-only project from one with an attached and synchronized drawing inventory.

---

# 2. Recovered Product Intent

## PI-AC0001-001 — ProjectStore is project intelligence, not merely tag storage

The conversation repeatedly expands ProjectStore from path storage and numbering into systems, tags, drawing membership, OPC relationships, audit/history, and Sheet Set integration.

## PI-AC0001-002 — AutoCAD tools should behave as one coherent project application

Binding, properties, tagging, OPCs, navigation, and batch operations are meant to share project context rather than operate as unrelated commands.

## PI-AC0001-003 — Project personnel configure engineering taxonomy without code changes

Systems, ranges, endpoints, and examples are intended to be modified by project users rather than embedded exclusively in C#.

## PI-AC0001-004 — ProjectStore should support intelligent P&ID relationships

The OPC model, page links, tag records, parent/child placement data, and drawing registry all indicate a move from isolated drafting entities toward governed engineering relationships.

## PI-AC0001-005 — Project bootstrap must support real project maturity stages

The system should not require a completed Sheet Set at the earliest drafting stage. It should allow progressive adoption while making capability readiness explicit.

---

# 3. Recovered Architectural Decision Candidates

These are **not yet accepted ADRs**. They should be compared with the current assessment and code.

## ADC-AC0001-001 — Canonical NOD binding with compatibility readers

Use one legal canonical store-path key for all new writes. Legacy bindings are compatibility inputs and should migrate toward the canonical representation.

## ADC-AC0001-002 — Separate interactive setup APIs from non-interactive runtime APIs

Setup/repair commands may prompt. Service and runtime paths must not unexpectedly invoke file dialogs.

## ADC-AC0001-003 — Separate project identity from store location

A project binding may include project ID/name while the canonical location resolver owns the store path. Mirrored values must not become independent authorities.

## ADC-AC0001-004 — Systems contain flexible category ranges

Logical form:

```text
System
└── Ranges
    ├── LINE
    ├── VALVE
    └── EQUIPMENT
```

Each category is optional per system.

## ADC-AC0001-005 — Unified engineering-tag registry

One project collection stores multiple tag categories using explicit category and system identity.

## ADC-AC0001-006 — Separate logical OPC connection from physical placements

One logical connection owns the shared OPC identity; two page-specific placements bind that logical connection to drawings/entities.

## ADC-AC0001-007 — Drawing registry uses stable drawing identity plus human page identity

The thread proposed stable drawing keys and a page-number index. Current architecture should preserve the distinction between durable drawing identity, path, fingerprint, and page number.

## ADC-AC0001-008 — Sheet Set is drawing-inventory authority; ProjectStore enriches it

The historical accepted direction was:

> The Sheet Set defines what drawings exist; ProjectStore defines what those drawings mean.

This requires reconciliation with the current governed authority model, which may distinguish native Sheet Set authority, project membership, and synchronized cache more precisely.

## ADC-AC0001-009 — Explicit project lifecycle/readiness states

Historical proposed states:

- BoundOnly
- SheetSetAttached
- Synced

The enduring decision is that project readiness must be explicit. Exact names remain subject to current design.

---

# 4. Architectural DNA Recovered

## DNA-AC0001-001 — Bind once, reuse everywhere

The user should establish project context once; tools consume it consistently.

## DNA-AC0001-002 — Authority before convenience

Do not let each command invent its own path, drawing list, tag counter, or endpoint source.

## DNA-AC0001-003 — Normal operation is deterministic and non-interactive

Routine engineering commands should not surprise users with configuration dialogs.

## DNA-AC0001-004 — Compatibility is directional

Legacy structures may be read and migrated, but new writes move toward the canonical architecture.

## DNA-AC0001-005 — Configuration belongs to the project domain

Systems and engineering ranges are project data, not hard-coded software constants.

## DNA-AC0001-006 — Logical identity is separate from graphical placement

Tags and OPCs exist as governed identities; drawing entities are placements/projections of those identities.

## DNA-AC0001-007 — Native authorities should be respected

Sheet Set data, AutoCAD drawings, and ProjectStore each have different native responsibilities. Synchronization should not collapse them into an accidental single blob.

## DNA-AC0001-008 — Progressive readiness over forced completeness

The product should support preliminary work while clearly gating features that require fuller project context.

## DNA-AC0001-009 — Conflicts are surfaced, not guessed away

Duplicate page numbers, missing pair placements, and competing bindings should become explicit conflicts.

---

# 5. Risks and Failure Modes Recovered

## RF-AC0001-001 — Illegal NOD key causes runtime failure

The original thread began because `/` in a dictionary key caused `eInvalidKey` and broke path resolution.

## RF-AC0001-002 — Multiple path-binding mechanisms create split-brain state

The conversation identified canonical root keys, nested project records, and legacy tag-store paths as parallel mechanisms.

## RF-AC0001-003 — Interactive overloads can leak into operational tools

A technically available prompt fallback may be called from the wrong context, producing repeated browsing and inconsistent bindings.

## RF-AC0001-004 — Mirrored store paths can drift

If project identity records and canonical path records both store the path without one owner, inconsistent values may emerge.

## RF-AC0001-005 — Human-editable JSON can drift in casing and shape

The conversation noted property-case inconsistency and redundant system identifiers. Validation and guided editing are required.

## RF-AC0001-006 — OPC data can conflate logical connection and placements

Storing only one record or using page data as identity would make cross-page reconciliation fragile.

## RF-AC0001-007 — Absolute paths are unstable project identity

The historical design discussion recognized the need for stable drawing keys and relative/project-aware identity.

## RF-AC0001-008 — Sheet Set sync can produce partial or ambiguous inventory

Duplicate page numbers, missing paths, and failed sync must be recorded as conflicts; partial writes should be avoided.

---

# 6. Future Vision Recovered

- Project-configurable systems and tag ranges.
- One project-wide tag allocation manager spanning lines, valves, and equipment.
- Off-page connector manager with cross-sheet matching and validation.
- Drawing/page navigation from project records.
- Sheet Set synchronization through full subset traversal.
- Project-wide audits and batch operations.
- Properties palette awareness of project context.
- Stable drawing registry enriched with fingerprints, status, and last-seen information.
- Append-style history and recovery/audit capabilities.
- Host-neutral shared synchronization logic with AutoCAD-specific adapters.

These are roadmap candidates, not evidence that implementation was completed.

---

# 7. Dead Ends or Superseded Approaches

## DE-AC0001-001 — Illegal slash-delimited NOD key

Must remain historical only. It is not a viable persistent key.

## DE-AC0001-002 — Repeated file browsing by ordinary tools

Contradicts the confirmed bind-once product intent.

## DE-AC0001-003 — Multiple independent path resolver classes as peer authorities

Legacy helpers may remain as adapters, but not as equal owners.

## DE-AC0001-004 — Treating `Systems` as merely object-category counters

The user explicitly clarified that systems are plant/process areas, while line/valve/equipment are categories beneath them.

## DE-AC0001-005 — Treating assistant-generated JSON as automatically canonical

The thread contains several evolving schemas. Their value is historical design evidence, not direct canonical promotion.

## DE-AC0001-006 — Immediate mandatory Sheet Set attachment at project creation

The user rejected forced immediate completeness because preliminary projects may not yet have a stable Sheet Set.

---

# 8. Open Questions for Human Review

1. Should starter systems live in a separate `SystemTemplates` collection, or should new projects seed editable active/disabled `Systems` records?
2. Is OPC number allocation globally unique for the project, unique per page, or represented by two levels: logical connection ID plus page-local display number?
3. Are exactly two OPC placements an absolute invariant, or may explicitly classified external/battery-limit endpoints have one internal placement plus one external endpoint record?
4. Does ProjectStore own project drawing membership, or does it cache/mirror membership from the active Sheet Set while preserving additional project-registration concepts?
5. Which of the historical project states remain appropriate after the current persistence convergence assessment?
6. Which historical tag fields are universal across LINE, VALVE, and EQUIPMENT, and which belong to type-specific extension records?
7. Should history be a complete append-only event log, a curated audit log, or both a current-state registry and event history?
8. Which historical code proposals were actually implemented and remain present in the current repository?

---

# 9. Promotion Recommendations

## PDR-001 candidates

- ProjectStore as project engineering authority.
- Bind-once/non-interactive runtime behavior.
- Process System domain definition.
- Flexible LINE/VALVE/EQUIPMENT ranges.
- Unified governed tag registry.
- OPC logical connection and placement model.
- Progressive project readiness.
- Sheet Set and drawing-registry roles.

## ADR candidates

- Canonical project binding and legacy compatibility.
- Interaction boundary: setup prompts versus runtime resolution.
- Sheet Set authority and ProjectStore synchronization.
- Logical OPC connection versus physical placements.
- Stable drawing identity and page-number indexing.
- Project readiness state model.

## Diagnostic/test candidates

- Detect all ProjectStore binding representations.
- Report competing or inconsistent path values.
- Validate system/range casing and category structure.
- Validate duplicate tag identities.
- Validate OPC pair cardinality and page references.
- Validate globally duplicate page numbers.
- Verify deterministic Sheet Set synchronization.

---

# 10. Mining Disposition

AC-0001 is a **high-value evidence artifact**. It contains direct human product intent, historical code, evolving schema proposals, and accepted architectural directions.

It should not be treated as one indivisible specification. Its strongest value is the recovery of enduring product rules and authority boundaries. The next step is human review of the open questions and promotion candidates, followed by comparison against the current `AutoCAD_Governed_Solution` assessment before updating PDR-001.
