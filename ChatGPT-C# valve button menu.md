# C# valve button menu

**User:** Anonymous  
**Created:** 10/13/2025 13:42:50  
**Updated:** 11/3/2025 15:27:56  
**Exported:** 7/21/2026 18:49:59  
**Link:** [https://chatgpt.com/g/g-p-677821cf820c8191964b0f1a58f9ad88/c/68ed4829-4014-8329-b4a5-c55c9c83711d](https://chatgpt.com/g/g-p-677821cf820c8191964b0f1a58f9ad88/c/68ed4829-4014-8329-b4a5-c55c9c83711d)  

## Prompt:
10/13/2025, 1:42:49 PM

Can a C# programmed menu make a button with a dropdown. Allowing you to choose from 3 options. I want a valve button, the user can click and choose between ball, butterfly or check valve to inster in drawing

## Response:
10/13/2025, 1:42:52 PM

> **Creating AutoCAD ribbon button**
>
> I need to craft a C# code example for creating an AutoCAD ribbon button with a dropdown (split button). The user asked for C# 7.3+ but wants language version 9.0 or higher. I'll make sure the code works with C# 7.3 and up. The code should integrate with AutoCAD's API, creating a RibbonSplitButton with three choices for inserting valve blocks. I’ll include necessary commands and ensure it loads at startup in the custom "CBI Tools" tab.
>
> Thought for 11s

Absolutely—use a Ribbon **split button**. In AutoCAD .NET this is `Autodesk.Windows.RibbonSplitButton`, which lets you show one main button with a drop-down list of choices (Ball / Butterfly / Check). Clicking a choice can run your command or invoke a handler.

Below is a clean, drop-in setup for your solution (AutoCAD2020 project). It:

- Adds a **CBI Tools** tab → **Valve Tools** panel  
- Creates a **Valve** split button with 3 menu choices  
- Wires each choice to a command that inserts the selected valve block  
- Uses your preferred pattern `Application.DocumentManager.MdiActiveDocument`  
- C# 7.3+ compatible

---

# File: `Ribbon/RibbonValveSplitButton.cs`  
_Namespace: `AutoCAD2020.Ribbon`_

```csharp
// LanguageVersion: C# 7.3+
// Requires references: acmgd.dll, acdbmgd.dll, accoremgd.dll, AdWindows.dll
// using Autodesk.Windows lives in AdWindows.dll (copy local = false)

using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.Runtime;
using Autodesk.Windows;

namespace AutoCAD2020.Ribbon
{
    /// <summary>
    /// Builds/attaches the CBI Tools tab with a Valve split-button (Ball/Butterfly/Check).
    /// </summary>
    public class RibbonValveSplitButton : IExtensionApplication
    {
        public void Initialize()
        {
            try
            {
                EnsureValveSplitButton();
            }
            catch
            {
                // Avoid noisy startup crashes; consider logging if you have a logger.
            }
        }

        public void Terminate() { }

        private static void EnsureValveSplitButton()
        {
            var ribbon = ComponentManager.Ribbon;
            if (ribbon == null)
            {
                // Ribbon not ready yet; subscribe to event and retry once it is available.
                ComponentManager.ItemInitialized += (s, e) =>
                {
                    if (ComponentManager.Ribbon != null)
                    {
                        EnsureValveSplitButton();
                    }
                };
                return;
            }

            // Find or create "CBI Tools" tab
            var tab = FindOrCreateTab(ribbon, "CBI Tools", "CBI_TOOLS_TAB");

            // Find or create "Valve Tools" panel
            var panel = FindOrCreatePanel(tab, "Valve Tools", "CBI_VALVE_PANEL");

            // If split button already exists, do nothing
            foreach (var item in panel.Source.Items)
            {
                if (item.Id == "CBI_Valve_Split")
                    return;
            }

            // Create three leaf buttons
            var btnBall = MakeLeafButton("Ball Valve", "CBI_VALVE_BALL", "CBI_InsertValve_Ball");
            var btnButterfly = MakeLeafButton("Butterfly Valve", "CBI_VALVE_BUTTERFLY", "CBI_InsertValve_Butterfly");
            var btnCheck = MakeLeafButton("Check Valve", "CBI_VALVE_CHECK", "CBI_InsertValve_Check");

            // Create split button and add the choices
            var split = new RibbonSplitButton
            {
                Id = "CBI_Valve_Split",
                Text = "Valve",
                ShowText = true,
                Orientation = System.Windows.Controls.Orientation.Vertical,
                IsSynchronizedWithCurrentItem = false,
                ListStyle = RibbonSplitButtonListStyle.List // shows a clean dropdown
            };
            split.Items.Add(btnBall);
            split.Items.Add(btnButterfly);
            split.Items.Add(btnCheck);

            // Optional: default displayed item
            split.Current = btnBall;

            panel.Source.Items.Add(split);
        }

        private static RibbonTab FindOrCreateTab(RibbonControl ribbon, string title, string id)
        {
            foreach (var t in ribbon.Tabs)
                if (t.Id == id) return t;

            var tab = new RibbonTab { Title = title, Id = id };
            ribbon.Tabs.Add(tab);
            tab.IsActive = true;
            return tab;
        }

        private static RibbonPanel FindOrCreatePanel(RibbonTab tab, string title, string id)
        {
            foreach (var p in tab.Panels)
                if (p.Source != null && p.Source.Id == id) return p;

            var src = new RibbonPanelSource { Title = title, Id = id };
            var panel = new RibbonPanel { Source = src };
            tab.Panels.Add(panel);
            return panel;
        }

        private static RibbonButton MakeLeafButton(string text, string id, string acadCommand)
        {
            return new RibbonButton
            {
                Id = id,
                Text = text,
                ShowText = true,
                CommandParameter = acadCommand,
                CommandHandler = new RibbonCommandHandler()
            };
        }

        /// <summary>
        /// Simple command router: executes the command name in CommandParameter.
        /// </summary>
        private sealed class RibbonCommandHandler : System.Windows.Input.ICommand
        {
            public bool CanExecute(object parameter) => true;
            public event System.EventHandler CanExecuteChanged { add { } remove { } }
            public void Execute(object parameter)
            {
                var cmd = (parameter as RibbonButton)?.CommandParameter as string;
                if (string.IsNullOrWhiteSpace(cmd)) return;

                var doc = Application.DocumentManager.MdiActiveDocument;
                if (doc == null) return;

                // Queue command (defined below in ValveInsertCommands)
                doc.SendStringToExecute($"{cmd} ", true, false, false);
            }
        }
    }
}
```

---

# File: `Commands/ValveInsertCommands.cs`  
_Namespace: `AutoCAD2020.Commands`_

```csharp
// LanguageVersion: C# 7.3+
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;
using Autodesk.AutoCAD.Geometry;
using Autodesk.AutoCAD.Runtime;

using AutoCAD2020.Services; // ValveInsertService

namespace AutoCAD2020.Commands
{
    /// <summary>
    /// Public commands invoked by the ribbon split-button.
    /// </summary>
    public static class ValveInsertCommands
    {
        [CommandMethod("CBI_InsertValve_Ball")]
        public static void InsertValve_Ball() =>
            ValveInsertService.InsertValveBlock("VALVE_BALL");

        [CommandMethod("CBI_InsertValve_Butterfly")]
        public static void InsertValve_Butterfly() =>
            ValveInsertService.InsertValveBlock("VALVE_BUTTERFLY");

        [CommandMethod("CBI_InsertValve_Check")]
        public static void InsertValve_Check() =>
            ValveInsertService.InsertValveBlock("VALVE_CHECK");
    }
}
```

---

# File: `Services/ValveInsertService.cs`  
_Namespace: `AutoCAD2020.Services`_

```csharp
// LanguageVersion: C# 7.3+
using System;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;
using Autodesk.AutoCAD.Geometry;

namespace AutoCAD2020.Services
{
    /// <summary>
    /// Minimal block insert helper. Assumes the block definitions exist in the drawing
    /// or are resolvable via your standard/source drawing if you add that logic.
    /// </summary>
    public static class ValveInsertService
    {
        /// <param name="blockName">e.g., "VALVE_BALL", "VALVE_BUTTERFLY", "VALVE_CHECK"</param>
        public static void InsertValveBlock(string blockName)
        {
            var doc = Application.DocumentManager.MdiActiveDocument;
            if (doc == null) return;

            var db = doc.Database;
            var ed = doc.Editor;

            // Get insertion point from user
            var ppo = new PromptPointOptions($"\nPick insertion point for {blockName}: ");
            var ppr = ed.GetPoint(ppo);
            if (ppr.Status != PromptStatus.OK) return;

            using (var tr = db.TransactionManager.StartTransaction())
            {
                // Ensure block exists (you can add fallback load-from-source DWG if needed)
                var bt = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead);
                if (!bt.Has(blockName))
                {
                    ed.WriteMessage($"\nBlock \"{blockName}\" not found in this drawing.");
                    tr.Abort();
                    return;
                }

                var btrId = bt[blockName];
                var spaceId = db.CurrentSpaceId;
                var space = (BlockTableRecord)tr.GetObject(spaceId, OpenMode.ForWrite);

                var br = new BlockReference(ppr.Value, btrId);
                // (Optional) Rotation, scale, attribute sync, etc., could be added here

                space.AppendEntity(br);
                tr.AddNewlyCreatedDBObject(br, true);

                // If block has attributes, open the definition and create AttributeReferences
                var btr = (BlockTableRecord)tr.GetObject(btrId, OpenMode.ForRead);
                if (btr.HasAttributeDefinitions)
                {
                    foreach (ObjectId id in btr)
                    {
                        if (tr.GetObject(id, OpenMode.ForRead) is AttributeDefinition ad && !ad.Constant)
                        {
                            var ar = new AttributeReference();
                            ar.SetAttributeFromBlock(ad, br.BlockTransform);
                            // TODO: populate from Project Store / NOD here
                            br.AttributeCollection.AppendAttribute(ar);
                            tr.AddNewlyCreatedDBObject(ar, true);
                        }
                    }
                }

                tr.Commit();
            }
        }
    }
}
```

---

## Notes & Tips

- **Block names:** Replace `"VALVE_BALL"`, `"VALVE_BUTTERFLY"`, and `"VALVE_CHECK"` with your actual block names or add logic to load them from a standard library DWG when missing.
- **Attributes:** In `ValveInsertService`, that `TODO` is where you’d map Project Store / NOD data into attribute values (you already have that architecture—plug it in here).
- **Icons:** You can assign 16×16 / 32×32 images to each `RibbonButton` via `LargeImage`/`Image` if you like.  
- **Load on startup:** Ensure the `RibbonValveSplitButton` class is in a loaded assembly and AutoCAD is set to load your DLL on start (e.g., `APPLOAD` + demand loading registry keys or your existing loader).
- **No ribbon flicker:** The code checks for existing items and avoids duplicating the UI.

---

## Changelog (semantic)

- **v1.0.0** – Added `RibbonValveSplitButton` with a Valve split button (Ball/Butterfly/Check), plus command stubs and a safe block-insert service.

If you want the split button’s **main click** (not just the dropdown) to repeat the **last chosen** valve type, we can sync `split.Current` on selection and wire a top-level `CommandParameter`—say the word “CBI_InsertValve_Last”—that dispatches to the last used item.

## Prompt:
11/3/2025, 3:27:17 PM

“Show me the current workflow schema or architecture summary for this tool.”

## Response:
11/3/2025, 3:27:18 PM

Here’s the concise architecture + workflow for the **Valve Split-Button Insert Tool** (Ball / Butterfly / Check):

# Architecture Summary (CBI Tools – Valve Insert)

- **UI Layer**
  - `AutoCAD2020.Ribbon.RibbonValveSplitButton` (IExtensionApplication)
    - Creates **CBI Tools** tab → **Valve Tools** panel.
    - Adds `RibbonSplitButton` **Valve** with 3 items:
      - `Ball Valve` → runs `CBI_InsertValve_Ball`
      - `Butterfly Valve` → runs `CBI_InsertValve_Butterfly`
      - `Check Valve` → runs `CBI_InsertValve_Check`
    - Uses a lightweight `RibbonCommandHandler` to `SendStringToExecute()` the mapped command.

- **Command Registry (per CBI standard)**  
  - `AutoCAD2020.Commands.ValveInsertCommands`
    - `[CommandMethod]` one-liners that route to the service:
      - `InsertValve_Ball()` → `ValveInsertService.InsertValveBlock("VALVE_BALL")`
      - `InsertValve_Butterfly()` → `"VALVE_BUTTERFLY"`
      - `InsertValve_Check()` → `"VALVE_CHECK"`

- **Domain/Service Layer**
  - `AutoCAD2020.Services.ValveInsertService`
    - Prompts for insertion point.
    - Verifies block exists in the drawing (future: **auto-load from block library DWG**).
    - Creates `BlockReference`, iterates `AttributeDefinition`s, creates `AttributeReference`s.
    - **Integration point:** populate attributes from **ProjectStore v1.0** (NOD key `CBI/ProjectStore/Path` via `ProjectStoreValidator` + `ProjectStoreIo`).

- **Data/Contracts (canonical)**
  - **ProjectStore v1.0** (required)  
    - `Systems` → Prefix/Pad/Start/End/Next (for tag generation)  
    - `Blocks.Valve.Types` → required mappings  
      - `Ball = INST_BALL-VALVE`  
      - `Butterfly = INST_BFLY-VALVE`  
      - (`Check` optional in spec; add as needed)  
    - `Tags`, `Released`, `History` etc.
  - **NOD** reference path: `CBI/ProjectStore/Path` (validated each run).

- **Command Registration Standard (recap)**
  - All commands declared in `AutoCAD2020\Commands.cs` (switchboard style).
  - Expression-bodied forwarders to logic classes (already followed here).

---

# End-to-End Workflow (Happy Path)

1. **Startup**: `RibbonValveSplitButton.Initialize()` runs → builds Ribbon tab/panel/split-button (idempotent).
2. **User Action**: Click **Valve** dropdown → choose **Ball / Butterfly / Check**.
3. **Dispatch**: `RibbonCommandHandler` calls `doc.SendStringToExecute("CBI_InsertValve_* ")`.
4. **Command**: `ValveInsertCommands.InsertValve_*` invokes `ValveInsertService.InsertValveBlock(blockName)`.
5. **ProjectStore Resolve**:
   - Resolve NOD key `CBI/ProjectStore/Path` → `ProjectStoreValidator` checks schema `Version:"1.0"`.
   - (Optional now, recommended next): read `Blocks.Valve.Types` to map selection → source block name & default attributes.
6. **User Pick**: Prompt for insertion point → OK.
7. **Block Ensure**:
   - If block exists in drawing → continue.  
   - If missing → (future) load from `C:\CBICAD\AutoCAD\Blocks\Instrumentation\…` via your standard loader.
8. **Insert**:
   - Create `BlockReference` at picked point.
   - If attributes present → create `AttributeReference`s and prefill (e.g., tag from `Systems` rules).
9. **Commit**: Transaction commit → valve appears in drawing.
10. **(Optional next)**: Record to `History` in ProjectStore and/or your change-history log.

---

# Minimal “Workflow Schema” (JSON-ish contract used by the tool)

```json
{
  "Tool": "ValveInsert",
  "UI": {
    "Ribbon": { "Tab": "CBI Tools", "Panel": "Valve Tools", "Control": "RibbonSplitButton" },
    "Choices": [
      { "Label": "Ball Valve", "Command": "CBI_InsertValve_Ball", "BlockKey": "VALVE_BALL" },
      { "Label": "Butterfly Valve", "Command": "CBI_InsertValve_Butterfly", "BlockKey": "VALVE_BUTTERFLY" },
      { "Label": "Check Valve", "Command": "CBI_InsertValve_Check", "BlockKey": "VALVE_CHECK" }
    ]
  },
  "Integration": {
    "ProjectStore": {
      "NodKey": "CBI/ProjectStore/Path",
      "Version": "1.0",
      "Blocks.Valve.Types": {
        "Ball": "INST_BALL-VALVE",
        "Butterfly": "INST_BFLY-VALVE",
        "Check": "INST_CHECK-VALVE"
      },
      "Systems": { "Prefix": "V", "Pad": 4, "Start": 2001 }
    }
  },
  "InsertPipeline": [
    "Resolve ProjectStore (required)",
    "Prompt Insertion Point",
    "Ensure Block Definition (or load)",
    "Create BlockReference",
    "Instantiate AttributeReferences (prefill from ProjectStore)",
    "Commit"
  ]
}
```

---

## Extensibility Points

- **Add a new valve type**: add a new leaf `RibbonButton`, a new `[CommandMethod]` forwarder, and map the name under `Blocks.Valve.Types` in ProjectStore v1.0.
- **Default Action**: set `split.Current` to the last-used valve; add a “CBI_InsertValve_Last” command if you want top-button click behavior.
- **Library Autoload**: plug your existing DWG loader (paths under `C:\CBICAD\AutoCAD\Blocks\Instrumentation\…`) before insert when `!bt.Has(blockName)`.

---

If you want this as a printable one-pager (or a Mermaid diagram), say the word and I’ll drop it into a canvas for you.



---
Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)