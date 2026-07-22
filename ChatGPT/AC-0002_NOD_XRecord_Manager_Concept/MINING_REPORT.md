# AC-0002 — Evidence Mining Report

## 1. Mining Status

**Status:** Mined — Awaiting Human Review  
**Source preserved:** Yes  
**Overall evidentiary value:** Moderate to High  
**Primary value:** Early product intent for a general-purpose drawing data inspector and manager

## 2. Source Boundary

This report is derived only from the preserved conversation in `SOURCE.md`.

The user explicitly proposed a tool that would expose the architecture of AutoCAD Named Objects Dictionary and XRecord data through a Sheet Set Manager-like interface. The assistant then expanded that concept with possible editing, cleanup, import/export, and implementation ideas. Those assistant expansions are retained as proposals, not treated as confirmed product requirements.

## 3. Recovered Product Intent

### PI-AC0002-001 — Make hidden drawing-resident application data visible

The tool is intended to let a user inspect data written by custom AutoCAD tools into the drawing's Named Objects Dictionary rather than requiring code-level inspection.

### PI-AC0002-002 — Present NOD architecture through a familiar management metaphor

The user explicitly correlated the proposed tool with AutoCAD's Sheet Set Manager. The intended interaction model is a navigable hierarchy rather than a raw diagnostic dump.

### PI-AC0002-003 — Reveal definition keys and their stored values

The core user need is visibility into dictionary keys and the data held beneath them. This is more specific than a generic XData viewer: the product should expose storage structure and contents together.

### PI-AC0002-004 — Support multiple custom subsystems

The concept arose while working on a layer-creation tool, but the user described inspecting data written by tools generally. The intended scope therefore appears broader than one subsystem.

## 4. Recovered Business Rules

### BR-AC0002-001 — NOD data must be traversable as a hierarchy

Nested `DBDictionary` structures must be represented recursively so that dictionary relationships are not flattened or lost.

**Evidence class:** Confirmed user intent plus directly aligned assistant interpretation.

### BR-AC0002-002 — XRecord values must retain their typed representation

A useful inspector must show the DXF code, interpreted type, and stored value for each `TypedValue` rather than displaying only a string conversion.

**Evidence class:** Assistant proposal strongly implied by the underlying data model; candidate requirement, not yet human-confirmed.

### BR-AC0002-003 — Inspection must identify both keys and data ownership location

The tool must communicate where a value exists in the dictionary hierarchy, not only the value itself.

**Evidence class:** Inference from the user's request to view the architecture and listed data.

### BR-AC0002-004 — The first implementation should not assume a single schema

Because the manager is intended to inspect data written by different tools, it should tolerate unknown dictionary and XRecord structures.

**Evidence class:** Architectural inference. Requires confirmation before promotion.

## 5. Recovered Architectural Decision Candidates

### ADC-AC0002-001 — Tree-and-detail-pane presentation

Use a two-pane interface:

- left: recursive dictionary and XRecord hierarchy;
- right: typed values or properties for the selected item.

This mirrors the user's Sheet Set Manager analogy and is the strongest UI candidate in the thread.

### ADC-AC0002-002 — Recursive dictionary traversal

A traversal service should recursively enumerate nested `DBDictionary` entries and classify leaf objects such as `Xrecord`.

### ADC-AC0002-003 — Separate inspection from mutation

Although the historical assistant proposed rename, delete, add, import, and edit functions, current repository governance suggests inspection should precede mutation. A read-only diagnostic mode is the safest initial boundary.

**Evidence class:** Present-day architectural interpretation informed by the historical proposal; not a recovered historical decision.

### ADC-AC0002-004 — Represent XRecord entries as typed rows

A detail model should preserve at least:

- dictionary path;
- record key;
- value order;
- DXF code;
- decoded type;
- raw or formatted value.

## 6. Architectural DNA Recovered

- Hidden system state should be inspectable.
- Internal storage should be understandable without reading source code.
- Familiar UI metaphors reduce the cost of learning technical tools.
- Hierarchy and type information are part of meaning.
- Diagnostic visibility should precede cleanup or repair.
- General infrastructure is more valuable than one-off subsystem viewers.

## 7. Risks and Failure Modes

### RISK-AC0002-001 — Editing arbitrary NOD data can corrupt drawings or application state

The source proposes editing, renaming, deleting, and creating records, but it does not define ownership, validation, transaction safety, backup, or rollback policies.

### RISK-AC0002-002 — Unknown custom objects may not be safe to interpret generically

The concept focuses on dictionaries and XRecords, but a real NOD may contain other object types or records owned by Autodesk or third-party applications.

### RISK-AC0002-003 — Raw DXF values may be misleading without schema context

Showing a code and value is useful, but it does not prove the semantic meaning of the field.

### RISK-AC0002-004 — Cleanup features may confuse absence with orphaning

A record cannot safely be classified as orphaned merely because the manager does not recognize it.

### RISK-AC0002-005 — The sample traversal code is conceptual, not production-safe

The historical snippet omits explicit transaction handling, document locking, object ownership rules, exception handling, disposal considerations, and non-XRecord object classification.

## 8. Future Capability Candidates

- Read-only recursive NOD browser
- XRecord typed-value inspector
- Dictionary-path search
- Export selected records to JSON or Markdown
- Schema-aware views for known CBI records
- Comparison of NOD structures between drawings
- Detection of conflicting ProjectStore binding records
- Controlled backup before mutation
- Schema-registered edit adapters
- Diagnostic integration with ProjectStore, XData, and drawing relationship tools

## 9. Dead Ends or Ideas Requiring Restraint

The following historical assistant suggestions should not be promoted directly:

- unrestricted deletion of unknown records;
- unrestricted renaming of dictionary keys;
- generic editing without a schema owner;
- generic import that writes arbitrary JSON into XRecords;
- treating all unrecognized data as unused or orphaned.

These ideas remain historically useful because they reveal the envisioned breadth, but they require strong governance and likely belong after a read-only implementation.

## 10. Relationship to Current Repository Assessment

This evidence strengthens the case for the read-only diagnostic work already identified in the governed solution assessment:

- NOD inspector;
- XData inspector;
- project binding diagnostics;
- schema classification;
- drawing relationship exploration.

It also explains the historical product motivation behind the existing `Export NOD to Excel` and related inspection utilities: the user wanted a visual, understandable representation of hidden drawing data.

## 11. Promotion Candidates

### PDR-001

Promote the following product intent:

- CBI should provide a general diagnostic interface for drawing-resident application data.
- The interface should expose hierarchy, keys, data types, and values.
- The interaction model may use a Sheet Set Manager-like tree-and-detail layout.

### Diagnostic Tooling Architecture

Promote these candidate boundaries:

- initial implementation is read-only;
- unknown records are preserved and labeled, not modified;
- typed values retain order and DXF codes;
- mutation requires schema-specific authorization and rollback support.

### Future ADR candidate

`ADR — Read-Only First Policy for NOD/XRecord Inspection`

## 12. Human Review Questions

1. Is the NOD/XRecord Manager still part of the intended product roadmap?
2. Should its first release be strictly read-only?
3. Should it inspect all NOD records or default to CBI-owned namespaces with an optional advanced view?
4. Is JSON export intended mainly for diagnostics, migration, backup, or all three?
5. Should the manager eventually edit known CBI schemas through schema-specific forms rather than raw typed-value editing?
6. Do you still envision it as a standalone palette, a modal form, or a tab within a broader Project Data Explorer?

## 13. Disposition

Preserve as early product-intent evidence. Promote the inspection and visualization concepts after human confirmation. Do not promote unrestricted mutation features or the sample traversal code as implementation requirements.
