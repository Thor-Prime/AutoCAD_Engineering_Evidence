# AC-0003 — Mining Report

## Disposition

Mined — Awaiting Human Review

## Executive Summary

This conversation began as a request for an all-in-one AutoCAD 2020 ribbon example, but it later captured a clearer user-interface architecture for the CBI application. The durable value is not the sample code itself. The durable value is the intent to present many independent AutoCAD tools as one coherent product through a runtime-generated `CBI Tools` ribbon, while preserving command registration as a separate switchboard.

The evidence also records several AutoCAD ribbon constraints: the public runtime API does not expose an absolute panel-width property; layout is influenced through columns, row breaks, item sizes, and slide-out content. These constraints should inform UI design but must be revalidated against the actual current ribbon implementation before code changes.

## Evidence Classification

### Confirmed Human Intent

1. The user wanted an AutoCAD 2020 C# ribbon example covering multiple control and layout types.
2. The user preferred to ignore XML/CUIX for the add-in workflow and continue with the C# runtime ribbon approach.
3. The user wanted a wider visible panel and specifically explored adding a third visible column containing four buttons.
4. The user accepted a 2×2 layout for the third column.
5. The user later requested a workflow/architecture summary for the ribbon tool.

### Direct Historical Observations

1. The historical design names a runtime tab `CBI Tools`.
2. The discussion groups command surfaces into panels such as `Home` and `Utilities`, with possible future groups including `Tags`, `QA/QC`, and `Sheet Sets`.
3. Commands named in the discussion include CBI Search, Find, OPC, Tag Manager, Layer Utility, XData Explorer, NOD Explorer, P&ID Sync, and Audit.
4. The summarized command convention keeps `[CommandMethod]` registrations in `Commands.cs` as a switchboard and lets ribbon controls invoke those registered command names.
5. The runtime ribbon uses `Autodesk.Windows` and `AdWindows.dll` rather than modifying a CUIX file.
6. A `RibbonRowPanel` and `RibbonRowBreak` were used as the proposed mechanism for visible column/row layout.
7. The evidence states that fixed absolute panel width is not exposed by the public ribbon API.

### Historical Assistant Proposals — Not Automatically Accepted

The following appeared as examples or recommendations and shall not be treated as current requirements without verification:

- The exact tab/panel/button names and arrangements in the kitchen-sink sample.
- A single central `switch`-based `RibbonCommandHandler` as the final command-dispatch architecture.
- Calling every command through `SendStringToExecute` rather than direct typed command/services.
- Automatically forcing the Ribbon visible by executing the `RIBBON` command.
- Removing and rebuilding panels by matching titles.
- Specific toggle, combo, textbox, slide-out, and contextual-tab behaviors.
- Specific deployment paths and autoload mechanisms.

## Recovered Business Rules

### BR-AC3-001 — The CBI toolset requires a coherent command surface

The user should experience the collection of CBI commands as one discoverable application rather than unrelated commands that must be memorized.

### BR-AC3-002 — Ribbon controls expose workflows, not implementation classes

A ribbon item should invoke a stable registered command or governed application action. The UI should not depend on internal class names.

### BR-AC3-003 — Command registration remains separate from UI composition

The historical summary explicitly describes `Commands.cs` as the command switchboard and the ribbon as a caller of those commands. This separation should be preserved unless a later governed decision replaces it.

### BR-AC3-004 — Tool grouping should reflect user workflow

Search, Find, OPC, Tag Manager, layer tools, diagnostics, synchronization, and audit were grouped according to how users discover and operate them, not according to assembly or namespace boundaries.

### BR-AC3-005 — Visible priority should be expressed through layout

Primary workflows may receive larger or more prominent controls, while secondary or optional functions may use compact grids, split controls, or slide-out areas.

### BR-AC3-006 — Ribbon initialization must not destabilize AutoCAD startup

Initialization failure should be contained and surfaced diagnostically rather than crashing the host application.

### BR-AC3-007 — Duplicate ribbon construction must be prevented

Reloading or reinitializing the add-in should not accumulate duplicate tabs or panels.

### BR-AC3-008 — AutoCAD layout constraints must be respected

The product should not depend on a fixed pixel width for ribbon panels because the historical investigation found that this is not available through the public runtime API.

## Recovered Product Intent

1. **Unified CBI application identity:** The `CBI Tools` tab acts as the visible shell for multiple subsystems.
2. **Discoverability:** Users should be able to locate functions through labeled panels and buttons rather than remembering command names.
3. **Workflow-oriented layout:** Search and project-data tools can be presented as primary actions; utilities and diagnostics can be grouped separately.
4. **Dynamic add-in deployment:** The runtime ribbon was preferred because it travels with the DLL and can be version-controlled with the application.
5. **Expandable interface:** The product was expected to grow into additional panels and richer controls as more tools became integrated.

## Architectural Decision Candidates

### ADC-AC3-001 — Runtime ribbon versus CUIX

Use the C# runtime ribbon API for the CBI add-in command surface, while recognizing CUIX as a separate mechanism more suitable for static enterprise UI customization.

**Status:** Strong historical direction; verify against current deployment architecture before promotion.

### ADC-AC3-002 — Commands as stable UI boundary

Ribbon actions should route through registered AutoCAD commands, keeping UI composition separate from feature implementation.

**Status:** Supported by the historical workflow summary and existing command-registration preference; candidate for promotion.

### ADC-AC3-003 — Idempotent ribbon construction

Ribbon initialization should create or reuse the tab and avoid duplicate panel accumulation.

**Status:** Candidate for implementation standard.

### ADC-AC3-004 — Workflow-based panels

Panel boundaries should represent user workflows or product capabilities rather than arbitrary code folders.

**Status:** Product/UI architecture candidate.

### ADC-AC3-005 — Layout through structural composition

Use columns, row breaks, item sizes, and optional slide-outs instead of relying on absolute panel width.

**Status:** Technical constraint and design guideline; revalidate with current Autodesk API/runtime behavior.

## Architectural DNA Recovered

1. **One product, many tools.**
2. **The UI should mirror the user's workflow.**
3. **Stable commands decouple interface from implementation.**
4. **Discoverability is a product requirement.**
5. **Primary actions deserve visible priority.**
6. **Host startup must remain resilient.**
7. **Initialization should be idempotent.**
8. **Work with platform constraints rather than fighting them.**
9. **UI architecture may evolve without rewriting the underlying commands.**

## Risks and Failure Modes

1. A large central command-handler `switch` can become another monolithic routing authority.
2. `SendStringToExecute` is stringly typed and may obscure failures, command availability, and execution ordering.
3. Rebuilding panels by title can be fragile if titles are localized or renamed.
4. Silently swallowing initialization exceptions can hide broken UI state unless reliable logging exists.
5. Forcing the Ribbon visible may override user workspace preferences.
6. A kitchen-sink control demonstration can be mistaken for an approved final product layout.
7. Ribbon grouping may drift away from product architecture as new tools are appended opportunistically.
8. Hardcoded command names can diverge from the canonical command registry.
9. Runtime-generated UI may not preserve user customization or rearrangement between loads.
10. `Autodesk.Windows`/`AdWindows.dll` integration is host-version-sensitive and must remain in the AutoCAD host project, not a platform-neutral shared project.

## Dead Ends or Superseded Approaches

1. Treating XML/CUIX as necessary for the CBI add-in ribbon was rejected for this historical workflow.
2. Attempting to assign an absolute pixel width to a runtime ribbon panel was found unsupported.
3. Treating the slide-out/fly-out as a normal visible column was corrected.
4. Using spacing hacks as a primary architectural layout mechanism should not be promoted.
5. The demonstration `Home`/`Utilities` kitchen-sink layout should not be mistaken for the canonical current ribbon.

## Future Vision Recovered

- Workflow-specific panels such as Tags, QA/QC, and Sheet Sets.
- Contextual tabs that respond to selection or project state.
- Galleries or combo controls for visual/configuration choices.
- State-aware toggles and input controls.
- Centralized icons and tooltip standards.
- Possible migration from string command dispatch to typed application actions where appropriate.
- A ribbon class map or manifest documenting tab → panel → column → row → item structure.

## Promotion Candidates

### PDR-001

- The CBI application should present a coherent, discoverable command surface.
- UI grouping should follow user workflows and product capabilities.
- The command surface should make the integrated-P&ID vision feel like one application rather than separate utilities.
- Primary workflows and secondary utilities should have intentionally different visual priority.

### ADR Candidates

- Runtime ribbon as the CBI add-in UI strategy.
- Command registry as the stable boundary between UI and features.
- Idempotent ribbon initialization.
- Host-only placement of Autodesk ribbon dependencies.

### Engineering Standard Candidates

- Ribbon IDs and command names must be centralized and unique.
- Ribbon initialization must log failures without crashing AutoCAD.
- Ribbon construction must be repeatable without duplicates.
- UI layouts must not rely on fixed panel widths.
- New buttons must reference an existing governed command/action rather than inventing an ad hoc execution route.

## Open Questions for Human Review

1. Is the runtime C# ribbon still the intended canonical command surface, or has the project since adopted a CUIX/bundle hybrid?
2. Should `Commands.cs` remain the sole `[CommandMethod]` switchboard?
3. Is `SendStringToExecute` still desired for ribbon invocation, or should the future ribbon use a typed command/action registry?
4. Which current tools are primary workflows versus utilities?
5. Should the ribbon be visible in every drawing, or should project-aware/contextual visibility be introduced?
6. Should the add-in force the Ribbon visible, or respect the user's workspace state?
7. Is preserving user ribbon rearrangement/customization a requirement?
8. Does a canonical current ribbon manifest already exist in the governed solution?

## Mining Conclusion

AC-0003 provides useful evidence of the intended CBI user experience: an integrated, discoverable runtime command surface that unifies many AutoCAD tools while keeping their command registration separate. It does not establish the exact current layout or implementation. The enduring contribution is the principle that the UI should communicate the product architecture and user workflow, while remaining resilient, idempotent, and decoupled from feature internals.
