# CBI ribbon organization

**User:** Anonymous  
**Created:** 9/11/2025 16:15:40  
**Updated:** 9/11/2025 16:43:58  
**Exported:** 7/21/2026 18:51:55  
**Link:** [https://chatgpt.com/g/g-p-677821cf820c8191964b0f1a58f9ad88/c/68c33bfb-2650-8323-b09b-212a386cf215](https://chatgpt.com/g/g-p-677821cf820c8191964b0f1a58f9ad88/c/68c33bfb-2650-8323-b09b-212a386cf215)  

## Prompt:
9/11/2025, 4:15:39 PM

CBI ribbon setup. In the conversation called "block attribute extraction". We discussed laying out the ribbon with a large icon and smaller buttons to the right. I want to take the time now and organize the menu with the projects I have.  I sent you the commands.cs code you refactored for me. Within that refactored code you can get the names of the programs I have made so far. Can you access this information i mention. Then let's discuss how to organize the tools in cluster or groups that make sence in a ribbon.

## Response:
9/11/2025, 4:15:41 PM

> **Organizing AutoCAD ribbon**
>
> We need to discuss how to organize the AutoCAD ribbon. Based on the user’s reference to "block attribute extraction" and related programs, we can only propose groups using known tools from the Model Set Context. For instance, utilities like Layer Utility, Block attribute tools, and Sheet Set Manager come to mind. We lack direct file access, though, so I can only work with the details provided.
>
> **Proposing ribbon layout**
>
> I can suggest organizing the Ribbon Tab "CBI Tools" with panels like Blocks, Layers, Data, Batch/Automation, Utilities, and Settings/Help. A large split button for primary actions with smaller buttons is a good idea. We can draft a ribbon layout, map the commands, and identify workflows with suggested icon sizes.
>
> Thought for 12s

I can’t directly open that older “block attribute extraction” chat or your `commands.cs` file from here. But I do remember a lot of the tools we’ve built and can give you a clean, sensible ribbon layout now—then we can drop your exact command names into the skeleton.

Below is a practical, tidy way to organize everything into one **CBI** tab with panels. It uses the “large icon left, small buttons stacked on the right” pattern you like (a `RibbonRowPanel` with one large `RibbonButton` + two or three small ones).

---

# Proposed CBI Ribbon Layout

## Tab: CBI Tools

### Panel: **Blocks & Attributes**
- **Large:** *Block Attribute Editor* — (e.g., `CBI-BlockAttribPalette` / `BlockAttribPropPalette`)
- **Small (row, right-side):**
  - *Apply to Current Drawing* — (`CBI-ApplyChangesIndividually`)
  - *Batch Update Toggle* — (`CBI-ToggleBatchMode`)
  - *Report Valve Links* — (`cbi-reportvalvelinks`)
- Optional flyout:
  - *Replace Block Symbol* — (your replace tool, or native `BLOCKREPLACE` as a helper)
  - *TextStyle Tools* — (`TextStyleForm` if still kept)

### Panel: **Layers**
- **Large:** *Layer Utility* — (`CBI-LayerUtility`)
- **Small:**
  - *Validate & Preview* — (`CBI-LayerUtilityValidation`)
  - *Import from Excel* — (`CBI-LayerExcelImporter`)
  - *Clean Unused (Discipline)* — (`CBI-LayerCleaner`)
- Optional flyout:
  - *Color/Lineweight Helpers* (debug/test commands)

### Panel: **Sheet Sets**
- **Large:** *Sheet Set Editor* — (`CBI-SheetSetEditorPalette`)
- **Small:**
  - *Load .DST* — (`CBI-LoadSheetSet`)
  - *List Sheets to Grid* — (`CBI-ListSheets`)
  - *Sync Block ↔ Sheet Props* — (`CBI-SyncBlockToSheetSet`)
- Optional flyout:
  - *Project Template Loader* (sheet set template bootstrap)

### Panel: **Data Explorer (NOD/XData)**
- **Large:** *CBI Data Explorer* — (`CBI-DataExplorerPalette`)  
  *(the tree/inspector we just made for NOD and, later, XData)*
- **Small:**
  - *View NOD* — (`CBI-NOD-View`)
  - *View XData* — (`CBI-XData-View`)
  - *Write NOD Entry* — (`CBI-NOD-Write`)
- Optional flyout:
  - *Change History Viewer* — (`CBI-ChangeHistory`)

### Panel: **Batch & Drawing Ops**
- **Large:** *Batch Runner* — (`CBI-BatchRun`)  
  *(process checked drawings; status checkbox column)*
- **Small:**
  - *Open Drawing* — (`CBI-OpenDrawingFile`)
  - *Switch to Model Space* — (`CBI-GoModelSpace`)
  - *Ensure Write Access* — (`CBI-EnsureWriteAccess`)
- Optional flyout:
  - *Lock/Unlock Diagnostics* — (`CBI-LockingValidation`)

### Panel: **Search & Utilities**
- **Large:** *CBI Search* — (`opencbisearch`)
- **Small:**
  - *Properties Palette* — (`CBI-Properties`)
  - *Notifications Test* — (`CBI-NotifyTest`)
  - *Settings* — (`CBI-Settings`)

### Panel: **Dev / Debug**
- **Large:** *Debug Console* — (`CBI-Debug`)
- **Small:**
  - *Dump Handles/Coords* — (`CBI-DumpSelection`)
  - *ResultBuffer Log* — (`CBI-LogRB`)
  - *Reload CBI DLL* — (`CBI-Reload`)

> If any of these names don’t match your actual command strings, no worries—we’ll slot in your exact names once you paste the latest `commands.cs`. The structure stays the same.

---

# Ribbon skeleton (C# 7.3; uses Autodesk Windows Ribbon)

This is a trimmed pattern you can paste into your `IExtensionApplication.Initialize()` flow. It creates the tab and one representative panel (repeat the pattern for each panel). It also shows the **large-left, small-right** layout with a `RibbonRowPanel`.

```csharp
// Using C# 7.3
using System;
using Autodesk.AutoCAD.Runtime;
using Autodesk.Windows;

[assembly: CommandClass(typeof(CBI.Ribbon.CBIRibbon))]

namespace CBI.Ribbon
{
    public class CBIRibbon : IExtensionApplication
    {
        public void Initialize()
        {
            try
            {
                EnsureCbiTab();
                AddBlocksAndAttributesPanel();   // repeat: AddLayersPanel(); AddSheetSetsPanel(); etc.
                // AddLayersPanel();
                // AddSheetSetsPanel();
                // AddDataExplorerPanel();
                // AddBatchPanel();
                // AddSearchUtilitiesPanel();
                // AddDevDebugPanel();
            }
            catch (Exception ex)
            {
                // swallow or log as needed
            }
        }

        public void Terminate() { }

        private RibbonTab EnsureCbiTab()
        {
            var rb = ComponentManager.Ribbon;
            if (rb == null) return null;

            var tab = FindTab("CBI_TOOLS_TAB");
            if (tab == null)
            {
                tab = new RibbonTab
                {
                    Title = "CBI Tools",
                    Id = "CBI_TOOLS_TAB"
                };
                rb.Tabs.Add(tab);
            }
            if (!tab.IsActive) tab.IsActive = true;
            return tab;
        }

        private RibbonTab FindTab(string id)
        {
            var rb = ComponentManager.Ribbon;
            if (rb == null) return null;
            foreach (var t in rb.Tabs)
            {
                if (t.Id == id) return t;
            }
            return null;
        }

        private void AddBlocksAndAttributesPanel()
        {
            var tab = EnsureCbiTab();
            if (tab == null) return;

            var panelSrc = new RibbonPanelSource
            {
                Title = "Blocks & Attributes",
                Id = "CBI_PANEL_BLOCKS_ATTR"
            };

            var panel = new RibbonPanel { Source = panelSrc };
            tab.Panels.Add(panel);

            // Left: Large primary button
            var btnLarge = MakeButton(
                text: "Attribute\nEditor",
                id: "CBI_BTN_BLOCKATTR_EDITOR",
                command: "CBI-BlockAttribPalette ",   // TODO: replace with your exact command name
                isLarge: true
            );

            // Right: small buttons stacked in a row panel
            var row = new RibbonRowPanel();

            var btnSmall1 = MakeButton(
                text: "Apply\nCurrent",
                id: "CBI_BTN_APPLY_CUR",
                command: "CBI-ApplyChangesIndividually ",
                isLarge: false
            );

            var btnSmall2 = MakeButton(
                text: "Batch\nToggle",
                id: "CBI_BTN_BATCH_TOGGLE",
                command: "CBI-ToggleBatchMode ",
                isLarge: false
            );

            var btnSmall3 = MakeButton(
                text: "Valve\nLinks",
                id: "CBI_BTN_REPORT_VALVE_LINKS",
                command: "cbi-reportvalvelinks ",
                isLarge: false
            );

            row.Items.Add(btnSmall1);
            row.Items.Add(btnSmall2);
            row.Items.Add(btnSmall3);

            panelSrc.Items.Add(btnLarge);
            panelSrc.Items.Add(row);
        }

        private RibbonButton MakeButton(string text, string id, string command, bool isLarge)
        {
            var btn = new RibbonButton
            {
                Text = text,
                ShowText = true,
                Id = id,
                CommandHandler = new AdskCommandHandler(command),
                Orientation = System.Windows.Controls.Orientation.Vertical,
                Size = isLarge ? RibbonItemSize.Large : RibbonItemSize.Standard,
                // TODO: assign Image/SmallImage from your PNGs (16x16 and 32x32)
            };
            return btn;
        }
    }

    // Simple command handler that feeds a command string to AutoCAD
    internal class AdskCommandHandler : System.Windows.Input.ICommand
    {
        private readonly string _cmd;
        public AdskCommandHandler(string cmd) => _cmd = cmd;

        public bool CanExecute(object parameter) => true;
        public event EventHandler CanExecuteChanged { add { } remove { } }

        public void Execute(object parameter)
        {
            try
            {
                // Send command with a trailing space to execute
                Autodesk.AutoCAD.ApplicationServices.Application
                    .DocumentManager
                    .MdiActiveDocument?
                    .SendStringToExecute(_cmd, true, false, true);
            }
            catch { /* optionally log */ }
        }
    }
}
```

**How to finish it quickly**
1. Keep the panel stubs (methods) one per panel: `AddLayersPanel()`, `AddSheetSetsPanel()`, etc., each identical to the sample but with different button sets.
2. Replace the placeholder command strings with your **actual** command names from `commands.cs`.
3. Wire your **32×32** (`Large`) and **16×16** (`Small`) PNGs into `RibbonButton.Image` / `SmallImage`.
4. Build to your existing output folder (`C:\CBICAD\Autocad`), and make sure the DLL loads at startup (your current setup).

If you paste your current `commands.cs` (or just the list of `[CommandMethod]` names), I’ll drop them into this layout and hand you the complete ribbon code with all panels filled out—no guesswork.

## Prompt:
9/11/2025, 4:17:42 PM

Here is the commands.cs this will give you a name of commands i currently have.

using Autodesk.AutoCAD.Runtime;
using System.Windows.Forms;
using Autodesk.AutoCAD.ApplicationServices;
using AutoCAD_Shared_Project.Interfaces;
using Autodesk.AutoCAD.Windows;
using AutoCAD2020.Palette;
using Autodesk.AutoCAD.DatabaseServices;
using AutoCAD2020.Objects;
using Autodesk.AutoCAD.EditorInput;
using AutoCAD;
using AutoCAD.Forms;
using AutoCAD.Sheet_Set_Manager;
using System;
using System.IO;
using ClosedXML.Excel;
using AutoCAD2020.Forms.Development;
using AutoCAD2020.CBI_Data_Explorer.UI;
//using static AutoCAD.Forms.TruckVectorBuilder;

namespace AutoCAD2020
{
    public class Commands

{
        [CommandMethod("CBI-DATA-EXPLORER", CommandFlags.Session)]
        public void ShowExplorer()
        {
            PaletteSetManager.Show();
        }

        [CommandMethod("CBI_ReportValveLinks")]
        public static void CBI_ReportValveLinks()
           => Services.ValveLinkReporter.Run();   // call the service logic you already have

        [CommandMethod("CBI_BlockAttribViewer")]
        public static void CBI_BlockAttribViewer()
            => Services.FormLauncher.ShowBlockAttribViewer();

        [CommandMethod("CBI_AddValveLabel")]
        public static void CBI_AddValveLabel()
            => Services.ValveLabelService.AddValveLabel();  // insert bubble + link to parent

        [CommandMethod("CBI_LinkValveTag")]
        public static void CBI_LinkValveTag()
            => Services.ValveTagService.LinkValveToBubbles(); // pick parent, then bubbles

        [CommandMethod("CBI_SyncBubbleTags")]
        public static void CBI_SyncBubbleTags()
            => Services.ValveTagService.SyncAllBubbles(); // resync TAG from VALVE_ID

        //// Launches the modeless dev form
        //[CommandMethod("CBI_BlockAttribViewer", CommandFlags.Modal)]
        //public static void CBI_BlockAttribViewer()
        //{
        //    var doc = Autodesk.AutoCAD.ApplicationServices.Application
        //        .DocumentManager.MdiActiveDocument;

        //    // IMPORTANT: reference your exact namespace for the form
        //    var form = new AutoCAD2020.Forms.Development.BlockAttributeGridForm();

        //    // Show modeless so user can keep working
        //    Autodesk.AutoCAD.ApplicationServices.Application
        //        .ShowModelessDialog(doc.Window.Handle, form, false);
        //}

        // (Optional) a ping command to verify commands are discoverable
        [CommandMethod("CBI_Ping")]
        public static void Ping()
        {
            var ed = Autodesk.AutoCAD.ApplicationServices.Application
                .DocumentManager.MdiActiveDocument.Editor;
            ed.WriteMessage("\nCBI commands are loaded and responding.");
        }

[CommandMethod("CBI_LayerMgr")]
        public void LyrMgr()
        {
            // Logic to open the OPC Form
            var layerMgr = new AutoCAD2020.Forms.LayerUtility();
            Autodesk.AutoCAD.ApplicationServices.Application.ShowModelessDialog(layerMgr);

            //AutoCAD.Forms.LayerUtility exporter = new AutoCAD.Forms.LayerUtility();
            //exporter.ExportGeometry();
        }

[CommandMethod("EXPORTGEOMETRY")]
        public void ExportGeometry()
        {
            AutoCAD.Forms.GeometryExporter exporter = new AutoCAD.Forms.GeometryExporter();
            exporter.ExportGeometry();
        }

private static PaletteSet _sheetSetPalette;

        [CommandMethod("CBI_SHEETSEDIT")]
        public void OpenSheetSetEditor()
        {
            if (_sheetSetPalette == null)
            {
                _sheetSetPalette = new PaletteSet("Sheet Set Editor")
                {
                    Style = PaletteSetStyles.ShowAutoHideButton |
                            PaletteSetStyles.ShowCloseButton |
                            PaletteSetStyles.Snappable,
                    DockEnabled = DockSides.Left
                };
                _sheetSetPalette.Size = new System.Drawing.Size(800, 600);
                _sheetSetPalette.Add("Editor", new SheetSetEditorPaletteControl());
            }

            _sheetSetPalette.Visible = true;
        }

[CommandMethod("DRAWTRUCKFULL")]
        public void DrawTruckFull()
        {
            Document doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
            Database db = doc.Database;

            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                BlockTable bt = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead);
                BlockTableRecord btr = (BlockTableRecord)tr.GetObject(bt[BlockTableRecord.ModelSpace], OpenMode.ForWrite);

                string csvPath = "C:\\CBICAD\\truck_full_outline.csv"; // Update this path if needed
                var lines = LineSegmentLoader.LoadLinesFromCsv(csvPath);
                LineSegmentLoader.InsertLinesToModelSpace(lines, btr, tr);

                tr.Commit();
            }
        }

        [CommandMethod("OpenCBISearch")]
        public void OpenCBISearchForm()
        {
            try

            {
                //// create an instance of the form using a factory method or direct call
                //ICBISearchForm form = CBISearchFormFactory.CreateInstance();
                ////CBI_Search form = new CBI_Search();
                //form.DisplayForm();
                var cbiForm = new CBI_Search();
                Autodesk.AutoCAD.ApplicationServices.Application.ShowModelessDialog(cbiForm);

            }
            catch (System.Exception ex)
            {
                MessageBox.Show("Error: " + ex.Message);
            }
        }

        [CommandMethod("OPENOPCFORM")]
        public void OpenOPCForm()
        {
            // Logic to open the OPC Form
            var opcForm = new OPC();
            Autodesk.AutoCAD.ApplicationServices.Application.ShowModelessDialog(opcForm);

        }

        [CommandMethod("OPENFINDFORM")]
        public void OpenFindForm()
        {
            // Logic to open the OPC Form
            var findForm = new Find();
            Autodesk.AutoCAD.ApplicationServices.Application.ShowModelessDialog(findForm);

        }

        [CommandMethod("TP_BLOCKMGR")]
        public void TPBlockMgr()
        {
            // Logic to open the OPC Form
            var findForm = new Find();
            Autodesk.AutoCAD.ApplicationServices.Application.ShowModelessDialog(findForm);

        }

[CommandMethod("SHOWBLOCKATTRIBPALETTE")]
        public static void ShowBlockAttribPalette()
        {
            if (_paletteSet == null)
            {
                _paletteSet = new PaletteSet("Block Attribute Editor");
                _sheetSetControl = new BlockAttribPropPaletteControl(_paletteSet); // Replace with your class name

                _paletteSet.Add("Attributes", _sheetSetControl);
                _paletteSet.Size = new System.Drawing.Size(300, 500);
                _paletteSet.DockEnabled = DockSides.Left | DockSides.Right;
            }

            _paletteSet.Visible = true;
        }

        private static PaletteSet _paletteSet;
        private static BlockAttribPropPaletteControl _sheetSetControl;

//[CommandMethod("CBIProperties")]
        //public void ShowCBIProperties()
        //{
        //    PropertiesPalette palette = new  PropertiesPalette();
        //    palette.Show();
        //}

        [CommandMethod("CBIProperties")]
        public void ShowCBIPropertiesWindow()
        {
            // Show the CBI Properties Palette once
            PropertiesPalette palette = new PropertiesPalette();
            palette.Show();

            // Subscribe to the SelectionAdded event to listen for object selection
            Document doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
            Editor ed = doc.Editor;

            ed.SelectionAdded += OnObjectSelected; // Event subscription
        }

        private void OnObjectSelected(object sender, SelectionAddedEventArgs e)
        {
            // Check if exactly one object is selected
            if (e.AddedObjects.Count == 1)
            {
                // Get the ObjectId of the selected object
                ObjectId selectedObjectId = e.AddedObjects[0].ObjectId;

                // Check if the object has pipe data attached
                PipeObject pipe = PipeDataHandler.GetPipeDataFromLine(selectedObjectId);

                if (pipe != null)
                {
                    // Update the properties palette with the pipe data
                    PropertiesPalette palette = new PropertiesPalette();
                    palette.UpdatePipeProperties(pipe); // Show pipe data in the Properties Palette
                }
                else
                {
                    // You can add some fallback behavior if no pipe data is found.
                    Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument.Editor.WriteMessage("\nNo pipe data found for the selected object.");
                }
            }
        }

[CommandMethod("AttachPipeData")]
        public void AttachPipeData()
        {
            Document doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
            Editor ed = doc.Editor;

            PromptEntityOptions options = new PromptEntityOptions("\nSelect a line to attach pipe data:");
            options.SetRejectMessage("\nOnly lines are allowed.");
            options.AddAllowedClass(typeof(Line), true);

            PromptEntityResult result = ed.GetEntity(options);

            if (result.Status == PromptStatus.OK)
            {
                ObjectId lineId = result.ObjectId;
                PipeObject pipe = new PipeObject(lineId)
                {
                    NominalDiameter = "150 mm",
                    FluidCode = "Water",
                    TagSequenceNumber = "001",
                    PipingMaterialClass = "A",
                    InsulationThickness = 10.0,
                    InsulationPurpose = "Thermal"
                };

                PipeDataHandler.AttachPipeDataToLine(lineId, pipe);
            }
        }

        [CommandMethod("ExportNODToExcel")]
        public static void ExportNODToExcel()
        {
            Document doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
            Database db = doc.Database;
            Editor ed = doc.Editor;

            string desktopPath = Environment.GetFolderPath(Environment.SpecialFolder.Desktop);
            string filePath = Path.Combine(desktopPath, "NOD_Export.xlsx");

            using (var workbook = new XLWorkbook())
            {
                var ws = workbook.Worksheets.Add("NOD Contents");
                ws.Cell(1, 1).Value = "Key";
                ws.Cell(1, 2).Value = "TypeCode";
                ws.Cell(1, 3).Value = "Value";

                int row = 2;

                using (Transaction tr = db.TransactionManager.StartTransaction())
                {
                    DBDictionary nod = tr.GetObject(db.NamedObjectsDictionaryId, OpenMode.ForRead) as DBDictionary;

                    foreach (DBDictionaryEntry entry in nod)
                    {
                        string key = entry.Key;
                        DBObject obj = tr.GetObject(entry.Value, OpenMode.ForRead);

                        if (obj is Xrecord xrec)
                        {
                            ResultBuffer rb = xrec.Data;
                            if (rb != null)
                            {
                                foreach (TypedValue tv in rb)
                                {
                                    ws.Cell(row, 1).Value = key;
                                    ws.Cell(row, 2).Value = tv.TypeCode;
                                    ws.Cell(row, 3).Value = tv.Value?.ToString() ?? "(null)";
                                    row++;
                                }
                            }
                            else
                            {
                                ws.Cell(row, 1).Value = key;
                                ws.Cell(row, 2).Value = "(none)";
                                ws.Cell(row, 3).Value = "(Xrecord has no data)";
                                row++;
                            }
                        }
                        else
                        {
                            ws.Cell(row, 1).Value = key;
                            ws.Cell(row, 2).Value = "(not Xrecord)";
                            ws.Cell(row, 3).Value = obj.GetType().Name;
                            row++;
                        }
                    }

                    tr.Commit();
                }

                workbook.SaveAs(filePath);
            }

            ed.WriteMessage($"\nNOD exported to Excel:\n{filePath}");
        }
    }
}

## Response:
9/11/2025, 4:17:44 PM

> **Organizing ribbon with C# 7.3**
>
> The goal is to organize the ribbon with the specified commands in C# 7.3. I'll build a layout using panels for each group of commands. Each button will correspond to a command, listed exactly as given, such as "CBI-DATA-EXPLORER", "CBI_BlockAttribViewer", "EXPORTGEOMETRY", and the rest. I'll provide the code skeleton for the row layout and include the layout for mapping the buttons. I'll also avoid suggesting browsing or tools.
>
> **Structuring ribbon with panels**
>
> I’ll organize the ribbon into categories like Valve Tagging Tools, Block & Attribute, and others, with each panel including specific buttons mapped to commands. I’ll provide a helper to generate buttons, give instructions for auto-loading, and ensure the ribbon is consistent at startup. I'll build a concise code that includes two fully defined panels and list the rest as mappings.
>
> Thought for 14s

Awesome—thanks for pasting `commands.cs`. I pulled the exact command names and laid out a clean **CBI** ribbon with sensible groups. It follows your preferred layout: a **large icon on the left** with **small buttons stacked on the right** (using a `RibbonRowPanel`). I also included ready-to-drop C# 7.3 code that wires five core panels. You can add the remainder with the quick mapping at the end.

---

# CBI Ribbon: panels & command mapping

## 1) Blocks & Attributes
- **Large:** Block Attribute Palette — `SHOWBLOCKATTRIBPALETTE`
- **Small (right):**
  - Attribute Viewer — `CBI_BlockAttribViewer`
  - Find (generic) — `OPENFINDFORM`
  - TP Block Mgr — `TP_BLOCKMGR` *(current stub opens Find form)*

## 2) Valve Tagging (Bubbles)
- **Large:** Add Valve Label — `CBI_AddValveLabel`
- **Small (right):**
  - Link Tag (parent↔bubbles) — `CBI_LinkValveTag`
  - Sync All Bubble Tags — `CBI_SyncBubbleTags`
  - Report Valve Links — `CBI_ReportValveLinks`

## 3) Layers
- **Large:** Layer Manager — `CBI_LayerMgr`
- **Small (right):**
  - Export Geometry — `EXPORTGEOMETRY`
  - (leave a placeholder small slot for your Layer Validation / Import when you expose those as commands)

## 4) Sheet Sets
- **Large:** Sheet Set Editor (palette) — `CBI_SHEETSEDIT`
- **Small (right):**
  - Open CBI Search — `OpenCBISearch`
  - OPC Form — `OPENOPCFORM`
  - Find — `OPENFINDFORM`

## 5) Data Explorer (NOD / XRecord now, XData later)
- **Large:** CBI Data Explorer — `CBI-DATA-EXPLORER`
- **Small (right):**
  - Export NOD → Excel — `ExportNODToExcel`
  - CBI Properties (selection-aware) — `CBIProperties`
  - Ping — `CBI_Ping`

## 6) Pipe Data (custom object)
- **Large:** Attach Pipe Data to Line — `AttachPipeData`
- **Small (right):**
  - CBI Properties — `CBIProperties` *(quick access)*
  - (future) View/Update pipe props *(when you publish commands)*

## 7) Geometry & Dev (optional panel or two)
- **Large:** Draw Truck Full — `DRAWTRUCKFULL`
- **Small (right):**
  - Ping — `CBI_Ping`
  - (future) Reload DLL / diagnostics (when you expose a command)

---

# Drop-in code (C# 7.3) — builds 5 core panels

> Put this in a `CBIRibbon.cs` (or similar) file in your AutoCAD2020 project.  
> It uses `SendStringToExecute` to call **your exact** command names from `commands.cs`.

```csharp
using System;
using Autodesk.AutoCAD.Runtime;
using Autodesk.Windows;

[assembly: CommandClass(typeof(CBI.Ribbon.CBIRibbon))]

namespace CBI.Ribbon
{
    public class CBIRibbon : IExtensionApplication
    {
        public void Initialize()
        {
            try
            {
                var tab = EnsureCbiTab();
                if (tab == null) return;

                AddBlocksAndAttributesPanel(tab);
                AddValveTaggingPanel(tab);
                AddLayersPanel(tab);
                AddSheetSetsPanel(tab);
                AddDataExplorerPanel(tab);

                // Optional: add more panels
                // AddPipeDataPanel(tab);
                // AddGeometryDevPanel(tab);
            }
            catch { /* swallow or log */ }
        }

        public void Terminate() { }

        private RibbonTab EnsureCbiTab()
        {
            var rb = ComponentManager.Ribbon;
            if (rb == null) return null;

            var tab = FindTab("CBI_TOOLS_TAB");
            if (tab == null)
            {
                tab = new RibbonTab
                {
                    Title = "CBI Tools",
                    Id = "CBI_TOOLS_TAB"
                };
                rb.Tabs.Add(tab);
            }
            if (!tab.IsActive) tab.IsActive = true;
            return tab;
        }

        private RibbonTab FindTab(string id)
        {
            var rb = ComponentManager.Ribbon;
            if (rb == null) return null;
            foreach (var t in rb.Tabs)
                if (t.Id == id) return t;
            return null;
        }

        // ---------- Panels ----------

        private void AddBlocksAndAttributesPanel(RibbonTab tab)
        {
            var panel = MakePanel(tab, "Blocks & Attributes", "CBI_PANEL_BLOCKS_ATTR");

            var btnLarge = MakeButton("Block\nAttrib Palette", "CBI_BTN_BLOCKATTR_PAL",
                "SHOWBLOCKATTRIBPALETTE ", isLarge: true);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeButton("Attrib\nViewer", "CBI_BTN_BLOCKATTR_VIEWER", "CBI_BlockAttribViewer ", false));
            row.Items.Add(MakeButton("Find", "CBI_BTN_FIND", "OPENFINDFORM ", false));
            row.Items.Add(MakeButton("TP Block\nMgr", "CBI_BTN_TPBLKMGR", "TP_BLOCKMGR ", false));

            panel.Source.Items.Add(btnLarge);
            panel.Source.Items.Add(row);
        }

        private void AddValveTaggingPanel(RibbonTab tab)
        {
            var panel = MakePanel(tab, "Valve Tagging", "CBI_PANEL_VALVE");

            var btnLarge = MakeButton("Add Valve\nLabel", "CBI_BTN_ADDVALVE", "CBI_AddValveLabel ", true);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeButton("Link\nTag", "CBI_BTN_LINKTAG", "CBI_LinkValveTag ", false));
            row.Items.Add(MakeButton("Sync\nBubbles", "CBI_BTN_SYNCBUB", "CBI_SyncBubbleTags ", false));
            row.Items.Add(MakeButton("Report\nLinks", "CBI_BTN_REPLINKS", "CBI_ReportValveLinks ", false));

            panel.Source.Items.Add(btnLarge);
            panel.Source.Items.Add(row);
        }

        private void AddLayersPanel(RibbonTab tab)
        {
            var panel = MakePanel(tab, "Layers", "CBI_PANEL_LAYERS");

            var btnLarge = MakeButton("Layer\nManager", "CBI_BTN_LAYERMGR", "CBI_LayerMgr ", true);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeButton("Export\nGeometry", "CBI_BTN_EXPORTGEOM", "EXPORTGEOMETRY ", false));
            // Placeholder(s) for upcoming layer tools when you expose them as commands:
            // row.Items.Add(MakeButton("Validate", "CBI_BTN_LYR_VALIDATE", "CBI_LayerValidate ", false));
            // row.Items.Add(MakeButton("Import\nExcel", "CBI_BTN_LYR_IMPORT", "CBI_LayerImport ", false));

            panel.Source.Items.Add(btnLarge);
            panel.Source.Items.Add(row);
        }

        private void AddSheetSetsPanel(RibbonTab tab)
        {
            var panel = MakePanel(tab, "Sheet Sets", "CBI_PANEL_SHEETS");

            var btnLarge = MakeButton("Sheet Set\nEditor", "CBI_BTN_SHEETEDIT", "CBI_SHEETSEDIT ", true);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeButton("CBI\nSearch", "CBI_BTN_CBISRCH", "OpenCBISearch ", false));
            row.Items.Add(MakeButton("OPC", "CBI_BTN_OPC", "OPENOPCFORM ", false));
            row.Items.Add(MakeButton("Find", "CBI_BTN_FIND2", "OPENFINDFORM ", false));

            panel.Source.Items.Add(btnLarge);
            panel.Source.Items.Add(row);
        }

        private void AddDataExplorerPanel(RibbonTab tab)
        {
            var panel = MakePanel(tab, "Data Explorer", "CBI_PANEL_DATAEX");

            var btnLarge = MakeButton("CBI Data\nExplorer", "CBI_BTN_EXPLORER", "CBI-DATA-EXPLORER ", true);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeButton("Export NOD\n→ Excel", "CBI_BTN_NOD2XLSX", "ExportNODToExcel ", false));
            row.Items.Add(MakeButton("CBI\nProperties", "CBI_BTN_CBIPROPS", "CBIProperties ", false));
            row.Items.Add(MakeButton("Ping", "CBI_BTN_PING", "CBI_Ping ", false));

            panel.Source.Items.Add(btnLarge);
            panel.Source.Items.Add(row);
        }

        // ---------- Helpers ----------

        private RibbonPanel MakePanel(RibbonTab tab, string title, string id)
        {
            var src = new RibbonPanelSource { Title = title, Id = id };
            var panel = new RibbonPanel { Source = src };
            tab.Panels.Add(panel);
            return panel;
        }

        private RibbonButton MakeButton(string text, string id, string command, bool isLarge)
        {
            return new RibbonButton
            {
                Text = text,
                ShowText = true,
                Id = id,
                Size = isLarge ? RibbonItemSize.Large : RibbonItemSize.Standard,
                Orientation = System.Windows.Controls.Orientation.Vertical,
                CommandHandler = new AdskCommandHandler(command),
                // TODO: assign Image (32x32) and SmallImage (16x16) PNGs for icons
            };
        }
    }

    internal class AdskCommandHandler : System.Windows.Input.ICommand
    {
        private readonly string _cmd;
        public AdskCommandHandler(string cmd) => _cmd = cmd;
        public bool CanExecute(object parameter) => true;
        public event EventHandler CanExecuteChanged { add { } remove { } }

        public void Execute(object parameter)
        {
            try
            {
                Autodesk.AutoCAD.ApplicationServices.Application
                    .DocumentManager.MdiActiveDocument?
                    .SendStringToExecute(_cmd, true, false, true);
            }
            catch { /* optional log */ }
        }
    }
}
```

---

## Add the remaining suggested panels quickly

If you want the **Pipe Data** and **Geometry/Dev** panels now, add these two methods and call them in `Initialize()`:

```csharp
private void AddPipeDataPanel(RibbonTab tab)
{
    var panel = MakePanel(tab, "Pipe Data", "CBI_PANEL_PIPEDATA");

    var btnLarge = MakeButton("Attach\nPipe Data", "CBI_BTN_ATTACHPIPE", "AttachPipeData ", true);

    var row = new RibbonRowPanel();
    row.Items.Add(MakeButton("CBI\nProperties", "CBI_BTN_CBIPROPS2", "CBIProperties ", false));
    // future: row.Items.Add(MakeButton("Update\nPipe", "CBI_BTN_PIPEUPDATE", "CBI_UpdatePipe ", false));

    panel.Source.Items.Add(btnLarge);
    panel.Source.Items.Add(row);
}

private void AddGeometryDevPanel(RibbonTab tab)
{
    var panel = MakePanel(tab, "Geometry & Dev", "CBI_PANEL_GEOMDEV");

    var btnLarge = MakeButton("Draw Truck\nFull", "CBI_BTN_TRUCK", "DRAWTRUCKFULL ", true);

    var row = new RibbonRowPanel();
    row.Items.Add(MakeButton("Ping", "CBI_BTN_PING2", "CBI_Ping ", false));
    // Add any future dev helpers here

    panel.Source.Items.Add(btnLarge);
    panel.Source.Items.Add(row);
}
```

---

## Notes & tips

- **Icons:** assign your 32×32 and 16×16 PNGs to `RibbonButton.Image` / `SmallImage`.  
- **Command strings:** I used *exact* names from your `commands.cs` (including case and underscores/dashes).  
- **Auto-load:** ensure this assembly loads on startup (per your `C:\CBICAD\Autocad` deployment).  
- **Future growth:** when you expose your Layer Validation/Import commands, just drop their small buttons into the **Layers** row.

If you want, I can wire **all** panels (including Pipe Data and Geometry/Dev) directly into a single file you can paste—just say the word and I’ll output the full class with every panel included.

## Prompt:
9/11/2025, 4:31:27 PM

Lets drop the drawtruckfull and ping.

Export to NOD is a tool for development. We replaced that with our CBI-Data-explorer. So let's not include that either. 

The category level tagging. This is a part of a bigger idea. Such that it will have a tool to insert a valve symbols. BALL, BUTTERFLY, OR GLOBE. Where the tag tool will add a tag bubble to either of the 3 dynamic visual state blocks. I plan to add more to this but for now I guess we group this as valves. The opencbisearch deals with blocks in the drawing it search a list of deg files for a criteria of names of blocks and lists result in a datagrid. Giving ability to export list to excel. We make edits and import the list for the program to find the blocks and make changes.

## Response:
9/11/2025, 4:31:28 PM

> **Updating ribbon plan**
>
> I need to revise the ribbon plan by removing buttons like CBI_Ping, DrawTruckFull, and ExportNODToExcel. We'll introduce a valve category with insert valve symbols (e.g., BALL/BUTTERFLY/GLOBE) and implement a tagging tool for dynamic visual state blocks. I’ll reorganize the panel with the necessary updates, including placeholders for valve insert functionality, and maintain existing commands for future implementation.
>
> **Creating placeholder buttons**
>
> Instead of defining new commands, it’s better to create buttons that either call existing commands or use placeholders. I’ll include stubs in the code with comments and instructions for valve insertion commands. We can use "Add Valve Label" for now, but I’ll add placeholders for "Insert Valve" with instructions for future development.
>
> **Updating ribbon panels**
>
> I’ll add buttons for the "InsertValve" functions, with tooltips like "define later" if the commands aren’t ready. The "Block Attribute" tools will move under the "Blocks & Search" panel, along with "OpenCBISearch". For the "Valves" panel, I'll use a RibbonSplitButton for the valve insert options and ensure everything follows the appropriate structure.
>
> Thought for 23s

Perfect—got it. I’ve removed **DrawTruckFull**, **Ping**, and **ExportNODToExcel**, and reworked the layout to put your **valve category/tagging** flow front-and-center. I also grouped **OpenCBISearch** with your block/attribute tools since it’s your block discovery → export/edit → re-import workflow.

Below is a ready-to-paste **C# 7.3** ribbon class that builds the updated panels. It only references commands you already have, and it leaves clean “hooks” for the upcoming **Insert Valve (BALL/BUTTERFLY/GLOBE)** commands (you can wire those later).

---

# Updated CBI Ribbon (no DrawTruckFull, no Ping, no NOD export)

**Panels in this version**
- **Valves** (category + tagging pipeline)
- **Blocks & Search** (block attribute palette/viewer + OpenCBISearch)
- **Layers**
- **Sheet Sets**
- **Data Explorer** (kept; trimmed to exclude NOD export)
- **Pipe Data** (attach pipe data + quick Properties)

```csharp
using System;
using Autodesk.AutoCAD.Runtime;
using Autodesk.Windows;

[assembly: CommandClass(typeof(CBI.Ribbon.CBIRibbon))]

namespace CBI.Ribbon
{
    public class CBIRibbon : IExtensionApplication
    {
        public void Initialize()
        {
            try
            {
                var tab = EnsureCbiTab();
                if (tab == null) return;

                AddValvesPanel(tab);           // NEW: centralized valve category + tagging
                AddBlocksAndSearchPanel(tab);  // Block attrib + OpenCBISearch
                AddLayersPanel(tab);
                AddSheetSetsPanel(tab);
                AddDataExplorerPanel(tab);     // trimmed (no ExportNODToExcel)
                AddPipeDataPanel(tab);         // attach/view pipe data
            }
            catch { /* optionally log */ }
        }

        public void Terminate() { }

        private RibbonTab EnsureCbiTab()
        {
            var rb = ComponentManager.Ribbon;
            if (rb == null) return null;

            var tab = FindTab("CBI_TOOLS_TAB");
            if (tab == null)
            {
                tab = new RibbonTab
                {
                    Title = "CBI Tools",
                    Id = "CBI_TOOLS_TAB"
                };
                rb.Tabs.Add(tab);
            }
            if (!tab.IsActive) tab.IsActive = true;
            return tab;
        }

        private RibbonTab FindTab(string id)
        {
            var rb = ComponentManager.Ribbon;
            if (rb == null) return null;
            foreach (var t in rb.Tabs)
                if (t.Id == id) return t;
            return null;
        }

        // --------------------------------------------------------------------
        // Panels
        // --------------------------------------------------------------------

        // 1) Valves: category-level tool + tagging pipeline
        private void AddValvesPanel(RibbonTab tab)
        {
            var panel = MakePanel(tab, "Valves", "CBI_PANEL_VALVES");

            // If your insert commands are not created yet, keep this split button—
            // once commands exist, just match the strings below.
            var insertSplit = new RibbonSplitButton
            {
                Text = "Insert Valve",
                IsSplit = true,
                Id = "CBI_BTN_VALVE_INSERT_SPLIT",
                // TODO: assign Image/SmallImage
            };
            insertSplit.Items.Add(MakeButton("BALL",      "CBI_BTN_VALVE_BALL",      "CBI_InsertValveBall ",    false));
            insertSplit.Items.Add(MakeButton("BUTTERFLY", "CBI_BTN_VALVE_BUTTERFLY", "CBI_InsertValveButterfly ", false));
            insertSplit.Items.Add(MakeButton("GLOBE",     "CBI_BTN_VALVE_GLOBE",     "CBI_InsertValveGlobe ",   false));

            // Large: current, concrete entry point in your tagging workflow
            var btnLarge = MakeButton("Add Valve\nLabel", "CBI_BTN_ADDVALVE", "CBI_AddValveLabel ", true);

            // Right stack: link, sync, report
            var row = new RibbonRowPanel();
            row.Items.Add(MakeButton("Link\nTag",    "CBI_BTN_LINKTAG",  "CBI_LinkValveTag ",    false));
            row.Items.Add(MakeButton("Sync\nBubbles","CBI_BTN_SYNCBUB",  "CBI_SyncBubbleTags ",  false));
            row.Items.Add(MakeButton("Report\nLinks","CBI_BTN_REPLINKS", "CBI_ReportValveLinks ",false));

            panel.Source.Items.Add(insertSplit);  // leftmost (acts like a large control)
            panel.Source.Items.Add(btnLarge);
            panel.Source.Items.Add(row);
        }

        // 2) Blocks & Search: Attribute palette/viewer + OpenCBISearch
        private void AddBlocksAndSearchPanel(RibbonTab tab)
        {
            var panel = MakePanel(tab, "Blocks & Search", "CBI_PANEL_BLOCKS_SEARCH");

            var btnLarge = MakeButton("Block\nAttrib Palette", "CBI_BTN_BLOCKATTR_PAL",
                "SHOWBLOCKATTRIBPALETTE ", true);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeButton("Attrib\nViewer", "CBI_BTN_BLOCKATTR_VIEWER", "CBI_BlockAttribViewer ", false));
            row.Items.Add(MakeButton("CBI\nSearch",    "CBI_BTN_CBISRCH",          "OpenCBISearch ",         false));
            row.Items.Add(MakeButton("Find",           "CBI_BTN_FIND",             "OPENFINDFORM ",          false));

            panel.Source.Items.Add(btnLarge);
            panel.Source.Items.Add(row);
        }

        // 3) Layers
        private void AddLayersPanel(RibbonTab tab)
        {
            var panel = MakePanel(tab, "Layers", "CBI_PANEL_LAYERS");

            var btnLarge = MakeButton("Layer\nManager", "CBI_BTN_LAYERMGR", "CBI_LayerMgr ", true);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeButton("Export\nGeometry", "CBI_BTN_EXPORTGEOM", "EXPORTGEOMETRY ", false));
            // Future smalls when you expose them as commands:
            // row.Items.Add(MakeButton("Validate", "CBI_BTN_LYR_VALIDATE", "CBI_LayerValidate ", false));
            // row.Items.Add(MakeButton("Import\nExcel", "CBI_BTN_LYR_IMPORT", "CBI_LayerImport ", false));

            panel.Source.Items.Add(btnLarge);
            panel.Source.Items.Add(row);
        }

        // 4) Sheet Sets
        private void AddSheetSetsPanel(RibbonTab tab)
        {
            var panel = MakePanel(tab, "Sheet Sets", "CBI_PANEL_SHEETS");

            var btnLarge = MakeButton("Sheet Set\nEditor", "CBI_BTN_SHEETEDIT", "CBI_SHEETSEDIT ", true);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeButton("OPC",  "CBI_BTN_OPC",  "OPENOPCFORM ",  false));
            row.Items.Add(MakeButton("Find", "CBI_BTN_FIND2","OPENFINDFORM ", false));

            panel.Source.Items.Add(btnLarge);
            panel.Source.Items.Add(row);
        }

        // 5) Data Explorer (no NOD export button)
        private void AddDataExplorerPanel(RibbonTab tab)
        {
            var panel = MakePanel(tab, "Data Explorer", "CBI_PANEL_DATAEX");

            var btnLarge = MakeButton("CBI Data\nExplorer", "CBI_BTN_EXPLORER", "CBI-DATA-EXPLORER ", true);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeButton("CBI\nProperties", "CBI_BTN_CBIPROPS", "CBIProperties ", false));

            panel.Source.Items.Add(btnLarge);
            panel.Source.Items.Add(row);
        }

        // 6) Pipe Data (attach + properties)
        private void AddPipeDataPanel(RibbonTab tab)
        {
            var panel = MakePanel(tab, "Pipe Data", "CBI_PANEL_PIPEDATA");

            var btnLarge = MakeButton("Attach\nPipe Data", "CBI_BTN_ATTACHPIPE", "AttachPipeData ", true);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeButton("CBI\nProperties", "CBI_BTN_CBIPROPS2", "CBIProperties ", false));

            panel.Source.Items.Add(btnLarge);
            panel.Source.Items.Add(row);
        }

        // --------------------------------------------------------------------
        // Helpers
        // --------------------------------------------------------------------

        private RibbonPanel MakePanel(RibbonTab tab, string title, string id)
        {
            var src = new RibbonPanelSource { Title = title, Id = id };
            var panel = new RibbonPanel { Source = src };
            tab.Panels.Add(panel);
            return panel;
        }

        private RibbonButton MakeButton(string text, string id, string command, bool isLarge)
        {
            return new RibbonButton
            {
                Text = text,
                ShowText = true,
                Id = id,
                Size = isLarge ? RibbonItemSize.Large : RibbonItemSize.Standard,
                Orientation = System.Windows.Controls.Orientation.Vertical,
                CommandHandler = new AdskCommandHandler(command),
                // TODO: assign Image (32x32) and SmallImage (16x16) PNGs for icons
            };
        }
    }

    internal class AdskCommandHandler : System.Windows.Input.ICommand
    {
        private readonly string _cmd;
        public AdskCommandHandler(string cmd) => _cmd = cmd;
        public bool CanExecute(object parameter) => true;
        public event EventHandler CanExecuteChanged { add { } remove { } }

        public void Execute(object parameter)
        {
            try
            {
                Autodesk.AutoCAD.ApplicationServices.Application
                    .DocumentManager.MdiActiveDocument?
                    .SendStringToExecute(_cmd, true, false, true);
            }
            catch { /* optionally log */ }
        }
    }
}
```

---

## Why this structure works

- **Valves panel** matches your bigger vision: a single home for symbol **insertion** and **tagging** (Add → Link → Sync → Report). The split-button groups BALL/BUTTERFLY/GLOBE cleanly and scales easily as you add more valve types.
- **Blocks & Search** keeps **OpenCBISearch** alongside the attribute editor/viewer so the “discover → edit → import back” loop stays in one place.
- **Data Explorer** remains your dev/operator window for NOD/XData/etc., without surfacing the now-redundant NOD export.
- **Pipe Data** is separate so your custom object work doesn’t clutter the tagging/search flows.

---

## Next quick steps

1. **Icons:** Add your 32×32 / 16×16 PNGs to each button (`Image` / `SmallImage`).
2. **Insert Valve commands:** When you add the actual commands (e.g., `CBI_InsertValveBall`), they’ll immediately work with the split button.
3. **Layer smalls:** When your validation/import/export are exposed as commands, drop them into the **Layers** row.

Want me to also output a **variant** where “Insert Valve” is the **large** button and “Add/Link/Sync/Report” are the smalls on the right? I can generate that version too.

## Prompt:
9/11/2025, 4:40:39 PM

I forgot I have a ribbonCommandHandler and a ribbonInitializer.cs would you like the code for those?

## Response:
9/11/2025, 4:40:40 PM

> **Responding to integration request**
>
> Yes, I’ll gladly accept the code! It’s great they’re offering it. I’ll suggest tailoring it to their existing handler interfaces, with methods like Execute and CanExecute. I'll provide a checklist of integration points, focusing on avoiding duplicates and managing reloads. While they paste the code, I’ll guide them through mapping it.
>
> Thought for 8s

Yes please—paste both `ribbonCommandHandler` and `ribbonInitializer.cs`.  

With those I’ll merge the layout we just designed into your existing wiring so you don’t have two competing initializers/handlers.

If it helps, here’s what I’ll align to your code when you share it:

- hook into your existing `Initialize()`/`CreateRibbon()` entry (no duplicate tabs)
- use your `ribbonCommandHandler` instead of my `AdskCommandHandler`
- match your icon loader (32×32 `Image`, 16×16 `SmallImage`) and resource paths
- keep your ID/string conventions for `RibbonTab`, `RibbonPanelSource`, and buttons
- preserve any reload-safe logic (e.g., removing old panels before re-adding)
- keep your exception/logging style

Drop them in and I’ll produce a clean, drop-in version.

## Prompt:
9/11/2025, 4:42:32 PM

using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.Windows;

namespace AutoCAD2020
{
    public class RibbonCommandHandler : System.Windows.Input.ICommand
    {
        public bool CanExecute(object parameter) => true;
        public event System.EventHandler CanExecuteChanged { add { } remove { } }

        public void Execute(object parameter)
        {
            // Autodesk ribbon passes the RibbonButton as the parameter
            var btn = parameter as RibbonButton;
            if (btn == null) return;

            var action = (btn.CommandParameter ?? string.Empty).ToString().Trim();
            if (string.IsNullOrEmpty(action)) return;

            var doc = Application.DocumentManager.MdiActiveDocument;

            switch (action.ToUpperInvariant())
            {
                // ---- Legacy aliases that map to your internal commands ----
                case "CBISEARCH":
                    doc.SendStringToExecute("opencbisearch ", true, false, false);
                    return;

                case "OPCFORM":
                    doc.SendStringToExecute("openopcform ", true, false, false);
                    return;

                case "FINDFORM":
                    doc.SendStringToExecute("openfindform ", true, false, false);
                    return;

                case "STYLECHANGE":
                    doc.SendStringToExecute("ShowTextStyleForm ", true, false, false);
                    return;
            }

            // ---- Generic fallback: treat CommandParameter as a real AutoCAD command ----
            // e.g., "CBI_BlockAttribViewer", "CBI_AddValveLabel", etc.
            doc.SendStringToExecute(action + " ", true, false, true);
        }
    }
}

//using Autodesk.AutoCAD.ApplicationServices;
//using Autodesk.Windows;
//using System.Windows.Input;

//namespace AutoCAD2020
//{

//    public class RibbonCommandHandler : System.Windows.Input.ICommand
//    {
//        public bool CanExecute(object parameter)
//        {
//            return true; // Always executable
//        }

//        public void Execute(object parameter)
//        {
//            if (parameter == null) return;

//            // Cast parameter to RibbonButton to access CommandParameter
//            var ribbonButton = parameter as RibbonButton;
//            if (ribbonButton == null) return;

//            // Determine action based on the parameter
//            string action = ribbonButton.CommandParameter?.ToString().ToUpper();

//            switch (action)
//            {
//                case "CBISEARCH":
//                    Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument
//                        .SendStringToExecute("opencbisearch ", true, false, false);
//                    break;

//                case "OPCFORM":
//                    Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument
//                        .SendStringToExecute("openopcform ", true, false, false);
//                    break;

//                case "FINDFORM":
//                    Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument
//                        .SendStringToExecute("openfindform ", true, false, false);
//                    break;

//                case "STYLECHANGE":
//                    Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument
//                        .SendStringToExecute("ShowTextStyleForm ", true, false, false);
//                    break;

//                default:
//                    Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument.Editor
//                        .WriteMessage($"\nUnknown command parameter: {action}");
//                    break;
//            }
//        }

//    public event System.EventHandler CanExecuteChanged;
//    }
//}

using System.IO;
using System.Windows;
using System.Windows.Media.Imaging;
using System.Drawing;
using Autodesk.AutoCAD.Runtime;
using Autodesk.Windows;
using AutoCAD.Properties; // Resources.resx

namespace AutoCAD2020.Ribbon
{
    public class RibbonInitializer : IExtensionApplication
    {
        public void Initialize() => CreateRibbon();
        public void Terminate() { }

        private static void CreateRibbon()
        {
            var rc = ComponentManager.Ribbon;
            if (rc == null) return;

            var tab = rc.FindTab("CBI_TAB");
            if (tab == null)
            {
                tab = new RibbonTab { Title = "CBI Tools", Id = "CBI_TAB" };
                rc.Tabs.Add(tab);
            }

            AddLogoPanel(tab);          // leftmost
            AddProcessToolsPanel(tab);  // existing
            AddBlockToolsPanel(tab);    // NEW third group
        }

        // ---------------- Panels ----------------

        private static void AddLogoPanel(RibbonTab tab)
        {
            if (HasPanel(tab, "CBI_LOGO")) return;

            var src = new RibbonPanelSource { Title = "      CB&I      ", Id = "CBI_LOGO" };
            var pnl = new RibbonPanel { Source = src };

            var logo = MakeLargeButton(
                text: "               ",
                commandParameter: "CBI_LOGO_NOP",    // harmless placeholder
                image: Resources.CB_I_RGB_64,
                tooltip: "CB&I"
            );
            logo.IsEnabled = false;
            logo.Width = 64;

            src.Items.Add(logo);
            tab.Panels.Insert(0, pnl);
        }

        private static void AddProcessToolsPanel(RibbonTab tab)
        {
            if (HasPanel(tab, "CBI_PROCESS")) return;

            var src = new RibbonPanelSource { Title = "Process Tools", Id = "CBI_PROCESS" };
            var pnl = new RibbonPanel { Source = src };
            var handler = new AutoCAD2020.RibbonCommandHandler();

            src.Items.Add(MakeLargeButton("CBI Search", "CBISEARCH", Resources.CBI_Search_3D_32x32, "Open CBI Search Form", handler));
            src.Items.Add(MakeLargeButton("OPC Tags", "OPCFORM", Resources.OPC_Man_32x32, "Open OPC Data Form", handler));
            src.Items.Add(MakeLargeButton("Find", "FINDFORM", Resources.CBI_Stick_Figure_32x32, "Open Find Form", handler));
            src.Items.Add(MakeLargeButton("Block / Attribute Manipulator", "SHOWBLOCKATTRIBPALETTE", Resources.block_prop_32x32, "Change block properties and attributes", handler));

            tab.Panels.Add(pnl);
        }

        // NEW: Block commands live here
        private static void AddBlockToolsPanel(RibbonTab tab)
        {
            if (HasPanel(tab, "CBI_BLOCKS")) return;

            var src = new RibbonPanelSource { Title = "Block Tools", Id = "CBI_BLOCKS" };
            var pnl = new RibbonPanel { Source = src };
            var handler = new AutoCAD2020.RibbonCommandHandler();

            // For these, we pass the *real* AutoCAD command names.
            src.Items.Add(MakeLargeButton("Block Attrib Viewer", "CBI_BlockAttribViewer", Resources.CBI_Stick_Figure_32x32, "Open attribute viewer (dev)", handler));
            src.Items.Add(MakeLargeButton("Add Valve Label", "CBI_AddValveLabel", Resources.block_prop_32x32, "Insert bubble + link to valve", handler));
            src.Items.Add(MakeLargeButton("Link Valve ↔ Tags", "CBI_LinkValveTag", Resources.Labl_Tag, "Link valve to existing bubbles", handler));
            src.Items.Add(MakeLargeButton("Sync Bubble Tags", "CBI_SyncBubbleTags", Resources.block_prop_32x32, "Copy VALVE_ID to bubble TAGs", handler));

            tab.Panels.Add(pnl);
        }

        // --------------- Helpers ---------------

        private static bool HasPanel(RibbonTab tab, string id) 
        {
            foreach (var p in tab.Panels)
                if (p.Source != null && p.Source.Id == id) return true;
            return false;
        }

        private static RibbonButton MakeLargeButton(string text, string commandParameter, Bitmap image, string tooltip, System.Windows.Input.ICommand handler = null)
        {
            return new RibbonButton
            {
                Text = text,
                ShowText = true,
                ShowImage = true,
                Size = RibbonItemSize.Large,
                ToolTip = tooltip,
                LargeImage = ToBitmapSource(image),
                CommandParameter = commandParameter,
                CommandHandler = handler ?? new AutoCAD2020.RibbonCommandHandler()
            };
        }

        private static BitmapSource ToBitmapSource(Bitmap bitmap)
        {
            using (var ms = new MemoryStream())
            {
                bitmap.Save(ms, System.Drawing.Imaging.ImageFormat.Png);
                ms.Position = 0;
                var bi = new BitmapImage();
                bi.BeginInit();
                bi.StreamSource = ms;
                bi.CacheOption = BitmapCacheOption.OnLoad;
                bi.EndInit();
                return bi;
            }
        }
    }
}

//using System;
//using System.Runtime.InteropServices;
//using System.Windows.Interop;
//using System.Windows.Media.Imaging;
//using System.Drawing; // for System.Drawing.Bitmap

//using Autodesk.AutoCAD.ApplicationServices;
//using Autodesk.Windows;
//using Autodesk.AutoCAD.Runtime;
//using AutoCAD2020;
//using AutoCAD.Properties;
//using System.Windows;
//using System.Windows.Forms;

//public class RibbonInitializer : IExtensionApplication
//{
//    public void Initialize()
//    {
//        CreateRibbon();
//    }

//    public void Terminate() { }

//    public static class ImageHelper
//    {
//        // Converts a GDI+ Bitmap to a WPF BitmapSource
//        public static BitmapSource ConvertToBitmapSource(Bitmap bitmap)
//        {
//            IntPtr hBitmap = bitmap.GetHbitmap();
//            try
//            {
//                return Imaging.CreateBitmapSourceFromHBitmap(
//                    hBitmap,
//                    IntPtr.Zero,
//                    Int32Rect.Empty,
//                    BitmapSizeOptions.FromEmptyOptions());
//            }
//            finally
//            {
//                // Clean up the HBitmap to avoid memory leaks
//                DeleteObject(hBitmap);
//            }
//        }

//        [DllImport("gdi32.dll", SetLastError = true)]
//        [return: MarshalAs(UnmanagedType.Bool)]
//        private static extern bool DeleteObject(IntPtr hObject);
//    }

//    private void CreateRibbon()
//    {
//        // Get the main RibbonControl
//        RibbonControl ribbonControl = ComponentManager.Ribbon;
//        if (ribbonControl == null) return; // Ribbon not available

//        // Check if our Tab already exists
//        RibbonTab cbiTab = ribbonControl.FindTab("CBI_TAB");
//        if (cbiTab != null) return;

//        // Create the Tab
//        cbiTab = new RibbonTab
//        {
//            Title = "CBI Tools",
//            Id = "CBI_TAB"
//        };
//        ribbonControl.Tabs.Add(cbiTab);

//        // --------------------------------------------------------------------
//        // 1. Create a panel for the Logo (place it at index 0)
//        // --------------------------------------------------------------------
//        RibbonPanelSource logoPanelSource = new RibbonPanelSource
//        {
//            Title = "      CB&I      " // Could be empty or "Logo"; space can help hide the name
//        };

//        RibbonPanel logoPanel = new RibbonPanel
//        {
//            Source = logoPanelSource
//        };

//        // Insert the logo panel at the far-left side (index 0)
//        cbiTab.Panels.Insert(0, logoPanel);

//        // --------------------------------------------------------------------
//        // 2. Add a “logo button” that doesn’t act like a button
//        // --------------------------------------------------------------------
//        var logoButton = new RibbonButton
//        {
//            ShowText = true,                // We only want to show the image
//            Text = "               ",
//            ShowImage = true,
//            Size = RibbonItemSize.Large,     // Ensure it’s large enough for the logo
//            IsEnabled = false,              // Optional: disable so it’s not clickable
//            ToolTip = "   CB&I Logo   "         // Hover text if you like
//        };

//        //// Convert the PNG resource to a WPF image
//        //logoButton.LargeImage = ImageHelper.ConvertToBitmapSource(Resources.CB_I_RGB_64);

//        //// Add to the logo panel
//        //logoPanelSource.Items.Add(logoButton);

//        // 1) Grab the bitmap from Resources
//        System.Drawing.Bitmap logoBitmap = Resources.CB_I_RGB_64;

//        // 2) Convert it to a WPF BitmapSource
//        var logoBitmapSource = ImageHelper.ConvertToBitmapSource(logoBitmap);

//        // 3) Assign it to the RibbonButton’s LargeImage
//        logoButton.LargeImage = logoBitmapSource;

//        logoButton.Width = 64;

//        // --------------------------------------------------------------------
//        // 3. Set the button’s image
//        // --------------------------------------------------------------------
//        // Option 1: Load from embedded resources or project resources (recommended)
//        //   If you have the image in your Resources.resx:
//        //   logoButton.LargeImage = Imaging.CreateBitmapSourceFromHBitmap(
//        //       Resources.CompanyLogo.GetHbitmap(),
//        //       IntPtr.Zero,
//        //       Int32Rect.Empty,
//        //       BitmapSizeOptions.FromEmptyOptions()
//        //   );

//        // Option 2: Load from external file (not recommended for deployment, 
//        //           but sometimes easier during development)
//        //           Provide a valid path to your .png/.jpg:
//        //
//        // logoButton.LargeImage = new BitmapImage(new Uri("C:\\Path\\To\\YourLogo.png"));

//        // For demonstration, we’ll comment out the “external file” approach:
//        // logoButton.LargeImage = new BitmapImage(new Uri("C:\\YourLogo.png"));

//        // Add this “logoButton” to the logo panel
//        logoPanelSource.Items.Add(logoButton);

//        // --------------------------------------------------------------------
//        // 4. Create another panel for your actual commands (Text Search)
//        // --------------------------------------------------------------------
//        RibbonPanelSource panelSource = new RibbonPanelSource
//        {
//            Title = "Process Tools"
//        };

//        RibbonPanel panel = new RibbonPanel
//        {
//            Source = panelSource
//        };

//        cbiTab.Panels.Add(panel);

//        // Prepare our command handler
//        var ribbonCommandHandler = new RibbonCommandHandler();

//        // Create the buttons
//        var cbiSearchButton = new RibbonButton
//        {
//            Text = "CBI Search",           
//            CommandHandler = ribbonCommandHandler,
//            CommandParameter = "CBISEARCH",
//            ShowText = true,
//            ShowImage = true,
//            Size = RibbonItemSize.Large,
//            ToolTip = "Open CBI Search Form",
//            LargeImage = ConvertToBitmapSource(Resources.CBI_Search_3D_32x32)
//        };

//        var opcFormButton = new RibbonButton
//        {
//            Text = "OPC Tags",
//            CommandHandler = ribbonCommandHandler,
//            CommandParameter = "OPCFORM",
//            ShowText = true,
//            ShowImage = true,
//            Size = RibbonItemSize.Large,
//            ToolTip = "Open OPC Data Form",
//            LargeImage = ConvertToBitmapSource(Resources.OPC_Man_32x32)
//        };

//        var findFormButton = new RibbonButton
//        {
//            Text = "Find",
//            CommandHandler = ribbonCommandHandler,
//            CommandParameter = "FINDFORM",
//            ShowText = true,
//            ShowImage = true,
//            Size = RibbonItemSize.Large,
//            ToolTip = "Open Find Form",
//            LargeImage = ConvertToBitmapSource(Resources.CBI_Stick_Figure_32x32)
//        };

//        var styleChangeButton = new RibbonButton
//        {
//            Text = "Block / Attribute Manipulator",
//            CommandHandler = ribbonCommandHandler,
//            CommandParameter = "SHOWBLOCKATTRIBPALETTE",
//            ShowText = true,
//            ShowImage = true,
//            Size = RibbonItemSize.Large,
//            ToolTip = "Change block properties and attributes fast.",
//            LargeImage = ConvertToBitmapSource(Resources.block_prop_32x32)
//        };

//        // Add them to the "Text Search" panel
//        panelSource.Items.Add(cbiSearchButton);
//        panelSource.Items.Add(opcFormButton);
//        panelSource.Items.Add(findFormButton);
//        panelSource.Items.Add(styleChangeButton);

//        // Method to convert System.Drawing.Bitmap to BitmapSource
//        BitmapSource ConvertToBitmapSource(System.Drawing.Bitmap bitmap)
//        {
//            using (var memory = new System.IO.MemoryStream())
//            {
//                bitmap.Save(memory, System.Drawing.Imaging.ImageFormat.Png);
//                memory.Position = 0;
//                var bitmapImage = new BitmapImage();
//                bitmapImage.BeginInit();
//                bitmapImage.StreamSource = memory;
//                bitmapImage.CacheOption = BitmapCacheOption.OnLoad;
//                bitmapImage.EndInit();
//                return bitmapImage;
//            }
//        }
//    }
//}

## Response:
9/11/2025, 4:42:33 PM

> **Revisiting Ribbon integration**
>
> I need to integrate the provided RibbonCommandHandler and RibbonInitializer while keeping their layout in mind, like the Valves, Blocks & Search, and other specified panels. I'll update the RibbonInitializer to match their structure, still using the existing IDs and their Resources images. Also, I'll reuse their helpers for buttons and ensure correct command names are used, avoiding ones to be removed. We may add the small button helper for consistency.
>
> **Updating code with modifications**
>
> I’ll output the revised code, updating AddProcessToolsPanel and AddBlockToolsPanel, and adding new methods for all the panels. I’ll make sure to use appropriate command strings for buttons, like "CBI_BlockAttribViewer." For the split button (Insert Valve), I'll add placeholders and comment them as TODO since the user plans to add them later. I'll also include a MakeSmallButton helper, using the available resources like CBI_Search_3D_32x32. I’ll keep the logo panel intact too.
>
> Thought for 10s

Awesome—thanks for sharing both files. I merged your preferred layout into **your** existing wiring (kept your `RibbonCommandHandler`, resources, tab ID `CBI_TAB`, logo panel, etc.). Below is a **drop-in replacement** for `RibbonInitializer.cs` that builds the panels we agreed on:

- Valves (category insert split + Add/Link/Sync/Report)  
- Blocks & Search (Block Attribute Palette + Viewer + OpenCBISearch + Find)  
- Layers (LayerMgr + ExportGeometry; room for upcoming smalls)  
- Sheet Sets (SheetSet Editor + OPC + Find)  
- Data Explorer (CBI Data Explorer + CBI Properties; **no NOD export**)  
- Pipe Data (Attach Pipe Data + CBI Properties)  

It uses your `RibbonCommandHandler` and your `Resources` bitmaps. Large-left + small-right is done with a `RibbonRowPanel`. You can paste this over your current `RibbonInitializer.cs`.

```csharp
using System.IO;
using System.Windows.Media.Imaging;
using System.Drawing;
using Autodesk.AutoCAD.Runtime;
using Autodesk.Windows;
using AutoCAD.Properties; // Resources.resx

namespace AutoCAD2020.Ribbon
{
    public class RibbonInitializer : IExtensionApplication
    {
        public void Initialize() => CreateRibbon();
        public void Terminate() { }

        private static void CreateRibbon()
        {
            var rc = ComponentManager.Ribbon;
            if (rc == null) return;

            var tab = rc.FindTab("CBI_TAB");
            if (tab == null)
            {
                tab = new RibbonTab { Title = "CBI Tools", Id = "CBI_TAB" };
                rc.Tabs.Add(tab);
            }

            AddLogoPanel(tab);                 // leftmost fixed logo
            AddValvesPanel(tab);               // category + tagging pipeline
            AddBlocksAndSearchPanel(tab);      // attrib palette/viewer + OpenCBISearch + Find
            AddLayersPanel(tab);
            AddSheetSetsPanel(tab);
            AddDataExplorerPanel(tab);         // no NOD export
            AddPipeDataPanel(tab);
        }

        // ---------------- Panels ----------------

        private static void AddLogoPanel(RibbonTab tab)
        {
            if (HasPanel(tab, "CBI_LOGO")) return;

            var src = new RibbonPanelSource { Title = "      CB&I      ", Id = "CBI_LOGO" };
            var pnl = new RibbonPanel { Source = src };

            var logo = MakeLargeButton(
                text: "               ",
                commandParameter: "CBI_LOGO_NOP",    // harmless placeholder
                image: Resources.CB_I_RGB_64,
                tooltip: "CB&I"
            );
            logo.IsEnabled = false;
            logo.Width = 64;

            src.Items.Add(logo);
            tab.Panels.Insert(0, pnl);
        }

        // 1) Valves — insert split + Add/Link/Sync/Report
        private static void AddValvesPanel(RibbonTab tab)
        {
            if (HasPanel(tab, "CBI_VALVES")) return;

            var src = new RibbonPanelSource { Title = "Valves", Id = "CBI_VALVES" };
            var pnl = new RibbonPanel { Source = src };
            var handler = new AutoCAD2020.RibbonCommandHandler();

            // Split for category insert (placeholders until commands exist)
            var insertSplit = new RibbonSplitButton
            {
                Text = "Insert Valve",
                Id = "CBI_BTN_VALVE_INSERT_SPLIT",
                IsSplit = true
            };
            insertSplit.Items.Add(MakeSmallButton("BALL", "CBI_InsertValveBall", Resources.block_prop_32x32, "Insert BALL valve", handler));
            insertSplit.Items.Add(MakeSmallButton("BUTTERFLY", "CBI_InsertValveButterfly", Resources.block_prop_32x32, "Insert BUTTERFLY valve", handler));
            insertSplit.Items.Add(MakeSmallButton("GLOBE", "CBI_InsertValveGlobe", Resources.block_prop_32x32, "Insert GLOBE valve", handler));

            // Large primary: Add Valve Label (current concrete entry point)
            var addValveLarge = MakeLargeButton("Add Valve\nLabel", "CBI_AddValveLabel", Resources.block_prop_32x32,
                                                "Insert bubble + link to valve", handler);

            // Right stack: Link, Sync, Report
            var row = new RibbonRowPanel();
            row.Items.Add(MakeSmallButton("Link\nTag", "CBI_LinkValveTag", Resources.Labl_Tag, "Link valve to existing bubbles", handler));
            row.Items.Add(MakeSmallButton("Sync\nBubbles", "CBI_SyncBubbleTags", Resources.block_prop_32x32, "Sync TAGs from VALVE_ID", handler));
            row.Items.Add(MakeSmallButton("Report\nLinks", "CBI_ReportValveLinks", Resources.CBI_Stick_Figure_32x32, "Report valve↔bubble links", handler));

            src.Items.Add(insertSplit);
            src.Items.Add(addValveLarge);
            src.Items.Add(row);
            tab.Panels.Add(pnl);
        }

        // 2) Blocks & Search — palette/viewer + CBI Search + Find
        private static void AddBlocksAndSearchPanel(RibbonTab tab)
        {
            if (HasPanel(tab, "CBI_BLOCKS_SEARCH")) return;

            var src = new RibbonPanelSource { Title = "Blocks & Search", Id = "CBI_BLOCKS_SEARCH" };
            var pnl = new RibbonPanel { Source = src };
            var handler = new AutoCAD2020.RibbonCommandHandler();

            var paletteLarge = MakeLargeButton("Block\nAttrib Palette", "SHOWBLOCKATTRIBPALETTE",
                                               Resources.block_prop_32x32, "Change block properties and attributes", handler);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeSmallButton("Attrib\nViewer", "CBI_BlockAttribViewer", Resources.CBI_Stick_Figure_32x32, "Open attribute viewer", handler));
            row.Items.Add(MakeSmallButton("CBI\nSearch", "CBISEARCH", Resources.CBI_Search_3D_32x32, "Open CBI Search Form", handler));
            row.Items.Add(MakeSmallButton("Find", "FINDFORM", Resources.CBI_Stick_Figure_32x32, "Open Find Form", handler));

            src.Items.Add(paletteLarge);
            src.Items.Add(row);
            tab.Panels.Add(pnl);
        }

        // 3) Layers — LayerMgr + ExportGeometry (space left for upcoming validate/import)
        private static void AddLayersPanel(RibbonTab tab)
        {
            if (HasPanel(tab, "CBI_LAYERS")) return;

            var src = new RibbonPanelSource { Title = "Layers", Id = "CBI_LAYERS" };
            var pnl = new RibbonPanel { Source = src };
            var handler = new AutoCAD2020.RibbonCommandHandler();

            var layerLarge = MakeLargeButton("Layer\nManager", "CBI_LayerMgr",
                                             Resources.block_prop_32x32, "Open Layer Utility", handler);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeSmallButton("Export\nGeometry", "EXPORTGEOMETRY", Resources.block_prop_32x32,
                                          "Export geometry", handler));
            // Future:
            // row.Items.Add(MakeSmallButton("Validate", "CBI_LayerValidate", Resources.block_prop_32x32, "Validate layer sheet", handler));
            // row.Items.Add(MakeSmallButton("Import\nExcel", "CBI_LayerImport", Resources.block_prop_32x32, "Import layer definitions", handler));

            src.Items.Add(layerLarge);
            src.Items.Add(row);
            tab.Panels.Add(pnl);
        }

        // 4) Sheet Sets — Editor + OPC + Find
        private static void AddSheetSetsPanel(RibbonTab tab)
        {
            if (HasPanel(tab, "CBI_SHEETS")) return;

            var src = new RibbonPanelSource { Title = "Sheet Sets", Id = "CBI_SHEETS" };
            var pnl = new RibbonPanel { Source = src };
            var handler = new AutoCAD2020.RibbonCommandHandler();

            var sheetLarge = MakeLargeButton("Sheet Set\nEditor", "CBI_SHEETSEDIT",
                                             Resources.block_prop_32x32, "Open Sheet Set Editor", handler);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeSmallButton("OPC", "OPCFORM", Resources.OPC_Man_32x32, "Open OPC Data Form", handler));
            row.Items.Add(MakeSmallButton("Find", "FINDFORM", Resources.CBI_Stick_Figure_32x32, "Open Find Form", handler));

            src.Items.Add(sheetLarge);
            src.Items.Add(row);
            tab.Panels.Add(pnl);
        }

        // 5) Data Explorer — CBI Data Explorer + CBI Properties (no NOD export)
        private static void AddDataExplorerPanel(RibbonTab tab)
        {
            if (HasPanel(tab, "CBI_DATAEX")) return;

            var src = new RibbonPanelSource { Title = "Data Explorer", Id = "CBI_DATAEX" };
            var pnl = new RibbonPanel { Source = src };
            var handler = new AutoCAD2020.RibbonCommandHandler();

            var explorerLarge = MakeLargeButton("CBI Data\nExplorer", "CBI-DATA-EXPLORER",
                                                Resources.block_prop_32x32, "View NOD/XData/etc.", handler);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeSmallButton("CBI\nProperties", "CBIProperties", Resources.block_prop_32x32, "Selection-aware properties", handler));

            src.Items.Add(explorerLarge);
            src.Items.Add(row);
            tab.Panels.Add(pnl);
        }

        // 6) Pipe Data — attach + properties
        private static void AddPipeDataPanel(RibbonTab tab)
        {
            if (HasPanel(tab, "CBI_PIPEDATA")) return;

            var src = new RibbonPanelSource { Title = "Pipe Data", Id = "CBI_PIPEDATA" };
            var pnl = new RibbonPanel { Source = src };
            var handler = new AutoCAD2020.RibbonCommandHandler();

            var attachLarge = MakeLargeButton("Attach\nPipe Data", "AttachPipeData",
                                              Resources.block_prop_32x32, "Attach pipe data to a line", handler);

            var row = new RibbonRowPanel();
            row.Items.Add(MakeSmallButton("CBI\nProperties", "CBIProperties", Resources.block_prop_32x32, "View selected pipe data", handler));

            src.Items.Add(attachLarge);
            src.Items.Add(row);
            tab.Panels.Add(pnl);
        }

        // --------------- Helpers ---------------

        private static bool HasPanel(RibbonTab tab, string id)
        {
            foreach (var p in tab.Panels)
                if (p?.Source != null && p.Source.Id == id) return true;
            return false;
        }

        private static RibbonButton MakeLargeButton(string text, string commandParameter, Bitmap image, string tooltip,
                                                    System.Windows.Input.ICommand handler = null)
        {
            return new RibbonButton
            {
                Text = text,
                ShowText = true,
                ShowImage = true,
                Size = RibbonItemSize.Large,
                ToolTip = tooltip,
                LargeImage = ToBitmapSource(image),
                CommandParameter = commandParameter,
                CommandHandler = handler ?? new AutoCAD2020.RibbonCommandHandler()
            };
        }

        private static RibbonButton MakeSmallButton(string text, string commandParameter, Bitmap image, string tooltip,
                                                    System.Windows.Input.ICommand handler = null)
        {
            return new RibbonButton
            {
                Text = text,
                ShowText = true,
                ShowImage = true,
                Size = RibbonItemSize.Standard,
                ToolTip = tooltip,
                Image = ToBitmapSource(image),
                CommandParameter = commandParameter,
                CommandHandler = handler ?? new AutoCAD2020.RibbonCommandHandler()
            };
        }

        private static BitmapSource ToBitmapSource(Bitmap bitmap)
        {
            using (var ms = new MemoryStream())
            {
                bitmap.Save(ms, System.Drawing.Imaging.ImageFormat.Png);
                ms.Position = 0;
                var bi = new BitmapImage();
                bi.BeginInit();
                bi.StreamSource = ms;
                bi.CacheOption = BitmapCacheOption.OnLoad;
                bi.EndInit();
                return bi;
            }
        }
    }
}
```

### Notes
- Your `RibbonCommandHandler` already maps `CBISEARCH`, `OPCFORM`, `FINDFORM` aliases and falls back to real AutoCAD commands. I used **aliases** for those three and **real command names** everywhere else (e.g., `CBI_AddValveLabel`, `CBI_BlockAttribViewer`, etc.).
- The **Insert Valve** split uses placeholder commands (`CBI_InsertValveBall`, `CBI_InsertValveButterfly`, `CBI_InsertValveGlobe`). When you add them, they’ll just work. Until then, you can leave them or comment those items out.
- I removed **DrawTruckFull**, **Ping**, and **ExportNODToExcel** per your request.

If you want me to also output a tiny `README` snippet for your repo (panel list + command table) or wire in specific 16×16 icons you have by name, say the word and I’ll add it.



---
Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)