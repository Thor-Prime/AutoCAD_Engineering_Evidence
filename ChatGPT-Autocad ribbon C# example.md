# Autocad ribbon C# example

**User:** Anonymous  
**Created:** 9/23/2025 18:04:38  
**Updated:** 11/3/2025 15:45:34  
**Exported:** 7/21/2026 18:48:50  
**Link:** [https://chatgpt.com/g/g-p-677821cf820c8191964b0f1a58f9ad88/c/68d32783-8800-8324-aaa6-0c533811fb4b](https://chatgpt.com/g/g-p-677821cf820c8191964b0f1a58f9ad88/c/68d32783-8800-8324-aaa6-0c533811fb4b)  

## Prompt:
9/23/2025, 6:04:36 PM

Can you search for code on the internet that applies to Autocad 2020 c# ribbon creation. I am looking for like an all in one example to show how to create all different kinds of layouts.

## Response:
9/23/2025, 6:04:39 PM

> **Searching for AutoCAD ribbon examples**
>
> I need to find up-to-date resources for AutoCAD 2020's C# ribbon creation, especially focusing on layout examples. I'll look up Autodesk .NET Ribbon API articles, exploring sources like Kean Walmsley's blog, Jeremy Tammik's posts, and GitHub repositories featuring sample code. I’ll do a search to gather and focus on the right materials.
>
> **Searching for AutoCAD 2020 C# ribbon examples**
>
> [Custom Menu and Ribbon with code.](https://forums.autodesk.com/t5/net-forum/custom-menu-and-ribbon-with-code/td-p/10511529)
> [Modify Ribbon - AutoCAD DevBlog](https://adndevblog.typepad.com/autocad/2012/07/modify-ribbon.html)
> [Adding custom controls to AutoCAD Ribbon interface ...](https://github.com/MadhukarMoogala/SimpleToggleButton)
> [Ribbon Elements (.NET)](https://help.autodesk.com/view/OARX/2020/PTB/?guid=GUID-01548FB4-7DE1-4025-B4C6-5E33D3530E15)
> [Customizing AutoCAD: Creating a Custom Ribbon Panel](https://www.youtube.com/watch?v=fW0WqLp6czo)
> [Pimp my AutoCAD or Revit Ribbon - The Building Coder](https://thebuildingcoder.typepad.com/blog/2011/02/pimp-my-autocad-or-revit-ribbon.html)
> [Adding a tab or a panel to the ribbon in AutoCAD products](https://www.autodesk.com/support/technical/article/caas/sfdcarticles/sfdcarticles/Adding-a-tab-or-a-panel-to-the-ribbon-in-AutoCAD-products.html)
> [The New RibbonBar API in AutoCAD 2009](https://through-the-interface.typepad.com/through_the_interface/2008/04/the-new-ribbonb.html)
> [How to add toggle button to AutoCAD ribbon using .NET ...](https://stackoverflow.com/questions/74433982/how-to-add-toggle-button-to-autocad-ribbon-using-net-development)
> [Displaying an AutoCAD ribbon tab contextually using .NET](https://keanw.com/2011/10/displaying-an-autocad-ribbon-tab-contextually-using-net.html)
> [Add a Button to Existing Ribbon Panel - Jeremy Tammik](https://jeremytammik.github.io/tbc/a/0894_exist_ribbon_panel.htm)
>
> **Finding ribbon code examples**
>
> I need to provide an "all-in-one" code example for AutoCAD layouts. I'll consult Autodesk's resources and ADN DevBlog posts, focusing on sample layouts like buttons, combo boxes, and contextual tabs. For clarity, I'll craft a concise C# example, while curating relevant links and summarizing what each covers.
>
> Thought for 14s

Absolutely-here’s a curated set of solid, *practical* references plus a drop-in “kitchen-sink” C# sample that shows how to build an AutoCAD 2020 ribbon tab with many control layouts (large/small buttons, split buttons, toggles, dropdown/gallery, text box, separators, slide-out, tooltips, command handler, and autoload via `IExtensionApplication`).

# Best references (hand-picked)

- **Official 2020 .NET Ribbon Elements docs** - hierarchy + sample of creating/populating a Ribbon object. Great for understanding the object model (`RibbonControl` → `RibbonTab` → `RibbonPanel`/`RibbonPanelSource` → `Items`). ([Autodesk Help](https://help.autodesk.com/view/OARX/2020/PTB/?guid=GUID-01548FB4-7DE1-4025-B4C6-5E33D3530E15&utm_source=chatgpt.com))  
- **ADN DevBlog: Modify Ribbon (C#)** - practical snippets (tabs, panels, buttons) and how to get the runtime ribbon. ([adndevblog.typepad.com](https://adndevblog.typepad.com/autocad/2012/07/modify-ribbon.html?utm_source=chatgpt.com))  
- **Kean Walmsley (Through the Interface): advanced controls**  
  - Text box on ribbon + notifying your app. ([keanw.com](https://keanw.com/2011/09/adding-a-textbox-to-autocads-ribbon-using-net.html?utm_source=chatgpt.com))  
  - Expanding text box variant. ([keanw.com](https://keanw.com/2011/09/adding-an-expanding-textbox-to-autocads-ribbon-using-net.html?utm_source=chatgpt.com))  
  - Contextual ribbon tabs (show tab by rule/state). ([keanw.com](https://keanw.com/2011/10/displaying-an-autocad-ribbon-tab-contextually-using-net.html?utm_source=chatgpt.com))
- **ADN DevBlog: Visual Styles-like Combo/Gallery** - pattern for a `RibbonGallery`/combobox style control. ([adndevblog.typepad.com](https://adndevblog.typepad.com/autocad/2012/05/create-ribbon-control-like-visual-styles-combo-box.html?utm_source=chatgpt.com))  
- **Toggle button example (C#)** - shows `RibbonToggleButton`. ([Stack Overflow](https://stackoverflow.com/questions/74433982/how-to-add-toggle-button-to-autocad-ribbon-using-net-development?utm_source=chatgpt.com))  
- **GitHub: SimpleToggleButton** - full sample project that registers a custom ribbon control. ([GitHub](https://github.com/MadhukarMoogala/SimpleToggleButton?utm_source=chatgpt.com))  
- **Background/Q&A threads** - differences between runtime ribbon API vs CUI API; simple add tab/panel/button examples. ([Autodesk Forums](https://forums.autodesk.com/t5/net-forum/autodesk-autocad-ribbon/td-p/4653889?utm_source=chatgpt.com))

---

# Drop-in “all-in-one” sample (AutoCAD 2020, C# 7.3 / LangVersion ≥ 9)

> Compile against .NET Framework used by AutoCAD 2020 (usually 4.7.2), reference:
> - `acdbmgd.dll`, `acmgd.dll` (AutoCAD .NET)
> - **`AdWindows.dll`** (for `Autodesk.Windows` ribbon runtime API)

This single file creates:
- Tab **“CBI Tools”**
- Panels: **Home**, **Utilities** (with *slide-out*)
- Controls: large & small buttons, **split button** (dropdown), **toggle button**, **combo/gallery-style** (simplified), **text box** (fires event), separators, and **tooltips**
- A central **command handler** that maps ribbon clicks to your commands (e.g., `opencbisearch`)
- Loads automatically on AutoCAD start via `IExtensionApplication.Initialize()`

```csharp
// File: RibbonInitializer.cs
// Target: AutoCAD 2020 (Runtime Ribbon API)
// Language: C# 7.3 (LangVersion 9.0 or greater as preferred)
// Version: 1.0.0
// Changelog:
//  - 1.0.0: Initial "kitchen-sink" ribbon example (tab, panels, rows, buttons, split, toggle, textbox, gallery-like, slide-out, tooltips)

using System;
using System.Linq;
using Autodesk.AutoCAD.Runtime;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.Windows;

[assembly: CommandClass(typeof(AutoCAD2020.RibbonInitializer))]

namespace AutoCAD2020
{
    /// <summary>
    /// Initializes a custom ribbon tab with a variety of controls.
    /// </summary>
    public class RibbonInitializer : IExtensionApplication
    {
        private const string TabId = "CBI_TOOLS_TAB";
        private const string TabTitle = "CBI Tools";
        private const string PanelHomeTitle = "Home";
        private const string PanelUtilTitle = "Utilities";

        public void Initialize()
        {
            try
            {
                EnsureRibbonVisible();
                var ribbon = ComponentManager.Ribbon;
                if (ribbon == null) return;

                // Create (or reuse) our tab
                var tab = ribbon.Tabs.FirstOrDefault(t => t.Id == TabId);
                if (tab == null)
                {
                    tab = new RibbonTab { Title = TabTitle, Id = TabId, IsActive = true };
                    ribbon.Tabs.Add(tab);
                }

                // HOME PANEL
                var homePanel = BuildHomePanel();
                AddOrReplacePanel(tab, homePanel);

                // UTILITIES PANEL (with slide-out content)
                var utilPanel = BuildUtilitiesPanel();
                AddOrReplacePanel(tab, utilPanel);

                tab.IsActive = true;
            }
            catch (Exception ex)
            {
                // Swallowing exceptions on init is common; optionally log to acad console:
                Application.DocumentManager.MdiActiveDocument?
                    .Editor?.WriteMessage($"\n[CBI Ribbon] Init error: {ex.Message}");
            }
        }

        public void Terminate()
        {
            // Optional: remove the tab on unload
            var ribbon = ComponentManager.Ribbon;
            var tab = ribbon?.Tabs.FirstOrDefault(t => t.Id == TabId);
            if (tab != null) ribbon.Tabs.Remove(tab);
        }

        #region Build Panels

        private static RibbonPanel BuildHomePanel()
        {
            var src = new RibbonPanelSource { Title = PanelHomeTitle };

            // Row 1: Large primary button + SplitButton (dropdown)
            var row1 = new RibbonRowPanel();

            var btnOpenSearch = CreateLargeButton(
                text: "CBI Search",
                showText: true,
                cmdParam: "CBISEARCH",
                tooltip: "Open CBI Search form (opencbisearch).");

            var split = new RibbonSplitButton
            {
                Text = "Run",
                ShowText = true,
                IsSplit = true,
                Size = RibbonItemSize.Large
            };
            split.Items.Add(CreateSmallButton("Find", "FINDFORM", "Open Find form (openfindform)."));
            split.Items.Add(CreateSmallButton("OPC", "OPCFORM", "Open OPC form (openopcform)."));
            split.Items.Add(CreateSmallButton("TagMgr", "TAGMGR", "Open Tag Manager (opentagmgr)."));

            row1.Items.Add(btnOpenSearch);
            row1.Items.Add(split);
            src.Items.Add(row1);

            // Separator
            src.Items.Add(new RibbonSeparator());

            // Row 2: Toggle + Combo (gallery-like) + Small buttons
            var row2 = new RibbonRowPanel();

            var tgl = new RibbonToggleButton
            {
                Text = "Batch",
                ShowText = true,
                Size = RibbonItemSize.Standard,
                CommandParameter = "TOGGLE_BATCH",
                CommandHandler = new RibbonCommandHandler()
            };
            tgl.ToolTip = CreateToolTip("Batch Mode", "Toggle batch update ON/OFF.");

            // Minimal combo/gallery-like (real galleries need images/templates; this is a stub)
            var combo = new RibbonCombo
            {
                Text = "Mode",
                ShowText = true,
                Size = RibbonItemSize.Standard
            };
            combo.Items.Add(new RibbonComboItem { Text = "Single" });
            combo.Items.Add(new RibbonComboItem { Text = "Batch" });
            combo.Current = combo.Items[0];

            // Two small action buttons
            row2.Items.Add(tgl);
            row2.Items.Add(combo);
            row2.Items.Add(CreateSmallButton("Apply", "APPLY", "Apply current settings."));
            row2.Items.Add(CreateSmallButton("Undo", "UNDO", "Open Undo panel."));

            src.Items.Add(row2);

            // TextBox row (fires on Enter)
            var row3 = new RibbonRowPanel();
            var txt = new RibbonTextBox
            {
                Text = "Search text",
                ShowText = true,
                Size = RibbonItemSize.Standard
            };
            txt.EnterPressed += (s, e) =>
            {
                var doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
                var content = txt.TextValue?.Trim() ?? string.Empty;
                if (!string.IsNullOrEmpty(content))
                    doc?.SendStringToExecute($"_CBIQUICKSEARCH \"{content}\" ", true, false, false);
            };
            row3.Items.Add(txt);
            src.Items.Add(row3);

            return new RibbonPanel { Source = src };
        }

        private static RibbonPanel BuildUtilitiesPanel()
        {
            var src = new RibbonPanelSource { Title = PanelUtilTitle };

            // Standard row with small buttons + separator
            var row = new RibbonRowPanel();
            row.Items.Add(CreateSmallButton("Layers", "LAYTOOLS", "Open Layer Utility."));
            row.Items.Add(CreateSmallButton("XData", "XDATAEXP", "Explore XData on selection."));
            row.Items.Add(new RibbonSeparator());
            row.Items.Add(CreateSmallButton("NOD", "NODEXP", "Explore Named Objects Dictionary."));

            src.Items.Add(row);

            // Slide-out (appears when panel is expanded)
            var slide = new RibbonPanelSource { Title = "Utilities (More)" };
            slide.Items.Add(CreateSmallButton("P&ID Sync", "PIDSYNC", "Link block attrs to Sheet Set props."));
            slide.Items.Add(CreateSmallButton("Audit", "AUDITCBI", "Run audit/cleanup helpers."));
            var slideOut = new RibbonPanelSlideOut { Source = slide };
            src.SlideOut = slideOut;

            return new RibbonPanel { Source = src };
        }

        #endregion

        #region Factory helpers

        private static RibbonButton CreateLargeButton(string text, bool showText, string cmdParam, string tooltip)
        {
            var btn = new RibbonButton
            {
                Text = text,
                ShowText = showText,
                Size = RibbonItemSize.Large,
                Orientation = System.Windows.Controls.Orientation.Vertical,
                CommandParameter = cmdParam,
                CommandHandler = new RibbonCommandHandler()
            };
            btn.ToolTip = CreateToolTip(text, tooltip);
            return btn;
        }

        private static RibbonButton CreateSmallButton(string text, string cmdParam, string tooltip)
        {
            var btn = new RibbonButton
            {
                Text = text,
                ShowText = true,
                Size = RibbonItemSize.Standard,
                CommandParameter = cmdParam,
                CommandHandler = new RibbonCommandHandler()
            };
            btn.ToolTip = CreateToolTip(text, tooltip);
            return btn;
        }

        private static RibbonToolTip CreateToolTip(string title, string content)
        {
            return new RibbonToolTip
            {
                Title = title,
                Content = content
            };
        }

        private static void AddOrReplacePanel(RibbonTab tab, RibbonPanel panel)
        {
            // Remove existing panel with the same title to allow reloading
            var existing = tab.Panels.FirstOrDefault(p => p.Source?.Title == panel.Source?.Title);
            if (existing != null) tab.Panels.Remove(existing);
            tab.Panels.Add(panel);
        }

        private static void EnsureRibbonVisible()
        {
            // Make sure the ribbon UI is available
            var rb = ComponentManager.Ribbon;
            if (rb == null)
            {
                // Force the Ribbon to load if user turned it off
                var doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
                doc?.SendStringToExecute("RIBBON ", true, false, false);
            }
        }

        #endregion
    }

    /// <summary>
    /// Central handler that maps ribbon clicks to AutoCAD commands.
    /// </summary>
    public class RibbonCommandHandler : System.Windows.Input.ICommand
    {
        public bool CanExecute(object parameter) => true;
        public event EventHandler CanExecuteChanged { add { } remove { } }

        public void Execute(object parameter)
        {
            // Autodesk ribbon passes the RibbonButton/Toggle as the parameter
            if (parameter is RibbonItem item)
            {
                var action = (item.CommandParameter ?? string.Empty).ToString().Trim();
                if (string.IsNullOrEmpty(action)) return;

                var doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;

                switch (action.ToUpperInvariant())
                {
                    // Your existing commands (map to your real commands)
                    case "CBISEARCH": doc.SendStringToExecute("opencbisearch ", true, false, false); break;
                    case "FINDFORM":  doc.SendStringToExecute("openfindform ", true, false, false);   break;
                    case "OPCFORM":   doc.SendStringToExecute("openopcform ",  true, false, false);   break;
                    case "TAGMGR":    doc.SendStringToExecute("opentagmgr ",   true, false, false);   break;
                    case "APPLY":     doc.SendStringToExecute("cbiapply ",     true, false, false);   break;
                    case "UNDO":      doc.SendStringToExecute("cbiundo ",      true, false, false);   break;
                    case "LAYTOOLS":  doc.SendStringToExecute("layerutility ", true, false, false);   break;
                    case "XDATAEXP":  doc.SendStringToExecute("cbixdataexp ",  true, false, false);   break;
                    case "NODEXP":    doc.SendStringToExecute("cbinodexp ",    true, false, false);   break;
                    case "PIDSYNC":   doc.SendStringToExecute("cbipidsync ",   true, false, false);   break;
                    case "AUDITCBI":  doc.SendStringToExecute("cbiaudit ",     true, false, false);   break;

                    // Special: toggle example (no command, just feedback)
                    case "TOGGLE_BATCH":
                        // Typically you’d set a static flag or fire a command:
                        doc.Editor?.WriteMessage("\nBatch Mode toggled.");
                        break;

                    default:
                        doc.Editor?.WriteMessage($"\n[CBI Ribbon] Unknown action: {action}");
                        break;
                }
            }
        }
    }
}
```

### Notes
- **Images/Icons:** You can assign `LargeImage`/`Image` (as `System.Windows.Media.ImageSource`) on `RibbonButton`/`RibbonToggleButton`. If you have 16×16 and 32×32 PNGs, wire them here.
- **True Gallery:** For a full `RibbonGallery` (thumbnails with categories, selection changed events), follow the ADN pattern shown in the Visual-Styles combo/gallery article. ([adndevblog.typepad.com](https://adndevblog.typepad.com/autocad/2012/05/create-ribbon-control-like-visual-styles-combo-box.html?utm_source=chatgpt.com))
- **Contextual tabs:** If you want the tab to appear only under certain conditions (e.g., when a block is selected), adapt Kean’s contextual sample. ([keanw.com](https://keanw.com/2011/10/displaying-an-autocad-ribbon-tab-contextually-using-net.html?utm_source=chatgpt.com))
- **Text box callbacks:** The enter-to-command approach above is based on the patterns from Kean’s posts. ([keanw.com](https://keanw.com/2011/09/adding-a-textbox-to-autocads-ribbon-using-net.html?utm_source=chatgpt.com))
- **Runtime vs CUI:** This example uses the **runtime ribbon API** (`Autodesk.Windows`) rather than editing a CUI file; see discussion of the two approaches here. ([Autodesk Forums](https://forums.autodesk.com/t5/net-forum/autodesk-autocad-ribbon/td-p/4653889?utm_source=chatgpt.com))
- **Object model refresher:** Autodesk’s 2020 docs page is your north star for the hierarchy and item types. ([Autodesk Help](https://help.autodesk.com/view/OARX/2020/PTB/?guid=GUID-01548FB4-7DE1-4025-B4C6-5E33D3530E15&utm_source=chatgpt.com))

If you want, I can tailor this to your exact **CBI ribbon** (merge into your existing `RibbonInitializer.cs`, wire your real commands, add your icons, and group items to match your “Tags” grouping).

## Prompt:
9/23/2025, 6:06:11 PM

Also is xml still used in menu creation?

## Response:
9/23/2025, 6:06:11 PM

Good question 👍  

Yes — **XML is still used in AutoCAD menu creation**, but it’s tied to the **CUIX system** (Custom User Interface files), not the runtime ribbon API.  

Here’s the breakdown:

---

### 1. **CUI/CUIX files (XML under the hood)**
- AutoCAD’s menu/ribbon customization files (`.CUIX`) are **zip containers** that include:
  - **XML** describing menus, ribbons, toolbars, accelerators, etc.
  - Resources (icons, bitmaps).
- If you export/unzip a `.CUIX`, you’ll see an `.XML` definition for the structure.  
- Autodesk intended this as the *official long-term* system for distributing UI changes (versus LISP menus or legacy `.MNU`/`.MNS`).

---

### 2. **Runtime Ribbon API (C# / `Autodesk.Windows`)**
- What you and I have been working with.
- Pure C# object model → build panels/tabs at load time.
- **Does not touch XML** at all.
- More dynamic: you can create/remove/change ribbon elements while AutoCAD runs, no need to manage `.CUIX` files.

---

### 3. **Which should you use?**
- **For custom .NET solutions like yours** → the runtime ribbon API is better:
  - Loads automatically with your DLL.
  - No need to distribute/manage `.CUIX` files.
  - Easier version control (just code).
- **For enterprise-wide standard menus** (e.g., company-wide drafting environment, with static toolbars/ribbons) → `.CUIX` is still the standard way to ship and update menus, since users can load it via the `CUI` editor.

---

✅ So:  
- XML is still *there* in `.CUIX` menus.  
- But when you build in C# (like your **CBI ribbon**), you’re bypassing XML and working with the runtime API.  

---

Do you want me to also show you how to **export/unzip a `.CUIX`** and peek at the XML structure? That way you’d see what a ribbon looks like in XML vs the C# runtime build.

## Prompt:
9/23/2025, 6:07:32 PM

So xml formatting is ignored once complied?

## Response:
9/23/2025, 6:07:32 PM

Almost 🙂 — let me break it down clearly:  

---

### 1. **CUIX = compiled container**  
- A `.CUIX` file is basically a **ZIP archive**.  
- Inside it are **XML definition files** (for menus, ribbons, toolbars) + resources (icons, etc.).  
- AutoCAD **parses the XML at load time** and builds the ribbon/menus in memory.  

So the XML isn’t *ignored* — it’s what drives the compiled `.CUIX`. But you don’t normally touch it directly; AutoCAD reads it, not you.

---

### 2. **Editing CUIX**  
- When you use the **CUI Editor** inside AutoCAD, you’re modifying that XML via a GUI.  
- If you unzip a `.CUIX`, you’ll see the raw XML (it’s ugly, but you can read it).  

---

### 3. **C# runtime ribbons (your case)**  
- If you build ribbons in C# using `Autodesk.Windows` (`ComponentManager.Ribbon`, `RibbonTab`, etc.), you’re bypassing the `.CUIX` entirely.  
- Your DLL builds the ribbon in memory at runtime — **no XML involved**.  
- AutoCAD doesn’t compile or cache XML for your C# ribbon — it just runs your code.  

---

✅ **So:**  
- For `.CUIX` menus: XML is parsed and used, not ignored.  
- For C# runtime ribbons: XML is irrelevant/ignored — your code directly builds the UI tree in memory.  

---

Would you like me to show you a side-by-side of:  
- What the **XML looks like** in a `.CUIX` (e.g., a `<RibbonPanel>` block),  
- Versus the **equivalent C# runtime code**?  

That might help you visualize why people stick with runtime code for add-ins but still use `.CUIX` for company standards.

## Prompt:
9/23/2025, 6:08:15 PM

Yes

## Response:
9/23/2025, 6:08:17 PM

> **Providing XML vs. C# code comparison**
>
> I’ll provide a side-by-side example showcasing the balance between XML and C# code, with version annotations for C# 7.3 (LangVersion 9+). The XML example will cover CUIX structure, and the C# counterpart will use Autodesk.Windows API calls. I’ll demonstrate panel, button, split button, toggle, dropdown, and command linkage, noting limitations with RibbonGallery and RibbonTextBox in CUIX.
>
> Thought for 5s

Awesome—here’s a clean side-by-side so you can *see* how a CUIX (XML) definition maps to the **runtime** C# ribbon API. The two examples below create the same things:

- a tab **“CBI Tools”**
- a panel **“Home”**
- a **large button** (“CBI Search”) that runs your `opencbisearch` command
- a **split button** with two child items
- a **toggle button** (“Batch”)

# 1) CUIX (XML) way
> This is the kind of XML you’ll find if you export/unzip a `.CUIX` (rename `.cuix` → `.zip`, then open the XML inside). AutoCAD reads this at load time.

```xml
<!-- Version: 1.0.0  |  Purpose: CUIX-style ribbon sample -->

<Customization xmlns="http://www.autodesk.com/Customization">
  <RibbonRoot>
    <Tabs>
      <RibbonTab Name="CBI Tools" Id="CBI_TOOLS_TAB">
        <RibbonPanels>
          <RibbonPanel RefId="CBI_HOME_PANEL" />
        </RibbonPanels>
      </RibbonTab>
    </Tabs>

    <Panels>
      <RibbonPanel Name="Home" Id="CBI_HOME_PANEL">
        <RibbonPanelSources>
          <RibbonPanelSource>
            <Rows>
              <!-- Row 1: Large primary button + SplitButton -->
              <RibbonRow>
                <RibbonCommandButton
                  Size="Large"
                  Text="CBI Search"
                  CommandName="CBI_OpenSearch"
                  ToolTipTitle="CBI Search"
                  ToolTipDescription="Open CBI Search form (opencbisearch)."
                  />
                <RibbonSplitButton Text="Run" Size="Large">
                  <Items>
                    <RibbonCommandButton Size="Standard" Text="Find" CommandName="CBI_OpenFind" />
                    <RibbonCommandButton Size="Standard" Text="OPC"  CommandName="CBI_OpenOPC"  />
                  </Items>
                </RibbonSplitButton>
              </RibbonRow>

              <!-- Row 2: Toggle -->
              <RibbonRow>
                <RibbonToggleButton
                  Size="Standard"
                  Text="Batch"
                  CommandName="CBI_ToggleBatch" />
              </RibbonRow>
            </Rows>
          </RibbonPanelSource>
        </RibbonPanelSources>
      </RibbonPanel>
    </Panels>
  </RibbonRoot>

  <!-- Commands referenced above -->
  <CommandList>
    <!-- Tip: These are “macros”. You can use ^C^C to cancel first, then run your command -->
    <Command Name="CBI_OpenSearch" Macro="^C^Copencbisearch " />
    <Command Name="CBI_OpenFind"   Macro="^C^Copenfindform " />
    <Command Name="CBI_OpenOPC"    Macro="^C^Copenopcform " />
    <!-- Toggles often map to macros or LISP; many teams point to a small helper command -->
    <Command Name="CBI_ToggleBatch" Macro="^C^Ccbibatchtoggle " />
  </CommandList>

  <!-- Optional icon mapping (simplified) -->
  <ImageList>
    <Image Item="CBI_OpenSearch" LargeImage="Icons\cbi32.png" SmallImage="Icons\cbi16.png" />
  </ImageList>
</Customization>
```

**What to notice**
- The **structure** is `RibbonRoot > Tabs > Panels > Rows > Items`.
- Buttons reference **commands by name** (`CommandName="CBI_OpenSearch"`), which resolve to **macros** in `<CommandList>`.
- Icons are linked via an image list or per-item attributes.

---

# 2) C# runtime (Autodesk.Windows) way
> Your DLL builds the same UI **in memory** at load time. No XML/CUIX needed.  
> **Target:** AutoCAD 2020, C# 7.3 (LangVersion ≥ 9.0)

```csharp
// File: RibbonInitializer.cs
// Version: 1.0.0
// Changelog:
//  - 1.0.0: Matches the CUIX example (tab, panel, large button, split, toggle)

using System.Linq;
using Autodesk.AutoCAD.Runtime;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.Windows;

[assembly: CommandClass(typeof(AutoCAD2020.RibbonInitializer))]

namespace AutoCAD2020
{
    public class RibbonInitializer : IExtensionApplication
    {
        private const string TabId = "CBI_TOOLS_TAB";

        public void Initialize()
        {
            EnsureRibbonVisible();

            var ribbon = ComponentManager.Ribbon;
            if (ribbon == null) return;

            // Create tab (or reuse if it exists)
            var tab = ribbon.Tabs.FirstOrDefault(t => t.Id == TabId)
                      ?? new RibbonTab { Id = TabId, Title = "CBI Tools" };

            if (!ribbon.Tabs.Contains(tab)) ribbon.Tabs.Add(tab);

            // Build "Home" panel
            var panel = BuildHomePanel();
            var existing = tab.Panels.FirstOrDefault(p => p.Source?.Title == "Home");
            if (existing != null) tab.Panels.Remove(existing);
            tab.Panels.Add(panel);

            tab.IsActive = true;
        }

        public void Terminate() { /* optional: remove tab on unload */ }

        private static RibbonPanel BuildHomePanel()
        {
            var src = new RibbonPanelSource { Title = "Home" };

            // Row 1
            var row1 = new RibbonRowPanel();
            row1.Items.Add(new RibbonButton
            {
                Text = "CBI Search",
                ShowText = true,
                Size = RibbonItemSize.Large,
                CommandParameter = "CBISEARCH",
                CommandHandler = new RibbonCommandHandler(),
                ToolTip = new RibbonToolTip { Title = "CBI Search", Content = "Open CBI Search form (opencbisearch)." }
            });

            var split = new RibbonSplitButton
            {
                Text = "Run",
                ShowText = true,
                Size = RibbonItemSize.Large,
                IsSplit = true
            };
            split.Items.Add(MakeBtn("Find", "FINDFORM"));
            split.Items.Add(MakeBtn("OPC", "OPCFORM"));
            row1.Items.Add(split);
            src.Items.Add(row1);

            // Row 2: toggle
            var row2 = new RibbonRowPanel();
            row2.Items.Add(new RibbonToggleButton
            {
                Text = "Batch",
                ShowText = true,
                Size = RibbonItemSize.Standard,
                CommandParameter = "TOGGLE_BATCH",
                CommandHandler = new RibbonCommandHandler()
            });
            src.Items.Add(row2);

            return new RibbonPanel { Source = src };
        }

        private static RibbonButton MakeBtn(string text, string param) =>
            new RibbonButton
            {
                Text = text,
                ShowText = true,
                Size = RibbonItemSize.Standard,
                CommandParameter = param,
                CommandHandler = new RibbonCommandHandler()
            };

        private static void EnsureRibbonVisible()
        {
            if (ComponentManager.Ribbon == null)
            {
                var doc = Application.DocumentManager.MdiActiveDocument;
                doc?.SendStringToExecute("RIBBON ", true, false, false);
            }
        }
    }

    // Central handler that maps ribbon clicks to AutoCAD commands
    public class RibbonCommandHandler : System.Windows.Input.ICommand
    {
        public bool CanExecute(object parameter) => true;
        public event System.EventHandler CanExecuteChanged { add { } remove { } }

        public void Execute(object parameter)
        {
            var item = parameter as RibbonItem;
            var action = (item?.CommandParameter ?? string.Empty).ToString().Trim();
            if (string.IsNullOrEmpty(action)) return;

            var doc = Application.DocumentManager.MdiActiveDocument;

            switch (action.ToUpperInvariant())
            {
                case "CBISEARCH": doc.SendStringToExecute("opencbisearch ", true, false, false); break;
                case "FINDFORM":  doc.SendStringToExecute("openfindform ", true, false, false);   break;
                case "OPCFORM":   doc.SendStringToExecute("openopcform ",  true, false, false);   break;

                case "TOGGLE_BATCH":
                    doc.Editor?.WriteMessage("\n[CBI] Batch mode toggled.");
                    break;
            }
        }
    }
}
```

**What to notice**
- Same hierarchy, but **you build objects directly** (`RibbonTab` → `RibbonPanel`/`RibbonPanelSource` → `RibbonRowPanel` → items).
- Instead of XML **macros**, you use a **command handler** that calls `SendStringToExecute(...)` (or your own .NET commands).
- Icons: assign `LargeImage` / `Image` on each `RibbonButton` if you want 32×32 / 16×16 PNGs.

---

## Quick pros/cons recap
- **CUIX/XML**
  - ✅ Great for company-wide, mostly static menus shipped to lots of users.
  - ✅ Editable with the CUI editor; lives outside code.
  - ⚠️ Macros only (unless they call into your .NET commands).
- **C# runtime**
  - ✅ Perfect for add-ins: versioned with your code, dynamic, easy to group controls logically.
  - ✅ No CUIX to manage; full control flow in C# (events, toggles, state).
  - ⚠️ You must create everything at runtime; if users disable your add-in, the tab disappears (by design).

If you want, I can expand this side-by-side to include:
- a **gallery/combobox** example,
- a **panel slide-out**,
- and **icons** (16×16 & 32×32) wiring—so it mirrors your “Tags” grouping exactly.

## Prompt:
9/23/2025, 6:14:46 PM

Ok ignoring the xml. Is there a way to set a panel width wide. I say in code we tried the minimum but what if we went larger and set a wider witdh?

## Response:
9/23/2025, 6:14:46 PM

Good eye 👀 — this is one of those “hidden gotchas” in the **runtime ribbon API**.  

---

### 🔹 What’s controllable
- A `RibbonPanelSource` hosts **rows**, and rows host items.  
- Each item (`RibbonButton`, `RibbonRowPanel`, etc.) has a **`Size`** (`Large` / `Standard` / `Small`), which affects how much space AutoCAD *tries* to allocate.  
- You can also insert **`RibbonSeparator`** or empty `RibbonRowPanel` to “pad” things.  

---

### 🔹 What’s **not** controllable
- **Absolute panel width** (e.g., “make this panel 300px wide”) is **not exposed** in the public API.  
- AutoCAD’s ribbon is WPF-based, but Autodesk wraps it — the container sizing is managed internally by AutoCAD.  
- The only “width knob” you get is via:
  - Item size (`Large` vs `Standard` vs `Small`).
  - Grouping of rows (more rows stack vertically = panel grows wider if forced).
  - Slide-out panel (`RibbonPanelSource.SlideOut`) → lets you “hide” extras until expanded.

---

### 🔹 Workarounds developers use
1. **Row grouping trick**  
   Add a `RibbonRowPanel` with multiple standard-size items side by side. That forces the panel wider because AutoCAD won’t wrap them.  
   ```csharp
   var row = new RibbonRowPanel();
   row.Items.Add(MakeBtn("A", "CMD_A"));
   row.Items.Add(MakeBtn("B", "CMD_B"));
   row.Items.Add(MakeBtn("C", "CMD_C"));
   panel.Source.Items.Add(row);
   ```
   → Panel becomes wide enough to host all 3.

2. **Spacer hack**  
   Insert an invisible “filler” control (some devs add a dummy `RibbonSeparator` or a disabled `RibbonButton` with blank text). It eats space.  

3. **Force slide-out**  
   If you want “extra wide” for optional tools, put them in the **slide-out panel**. When the user expands it, that section can stretch more freely.  

---

### 🔹 Bottom line
- You **cannot set a fixed panel width** like in WinForms/WPF (`panel.Width = 300`).  
- The ribbon layout engine owns sizing.  
- You *can* influence width by:
  - Packing more items side by side (row panels).  
  - Using `Large` item sizes.  
  - Adding spacers/separators.  
  - Offloading overflow into slide-out content.  

---

⚡ Question for you:  
Do you want me to put together a **“wide row” code sample** (e.g., 5 buttons in a single row to stretch the panel), so you can try it out in your CBI ribbon?

## Prompt:
9/23/2025, 6:16:41 PM

Ok so you mentioned autodesk always makes a fly out. What is the flyout is defined as column 2. And we need to add a column 3 to put our 4 buttons in.

## Response:
9/23/2025, 6:16:45 PM

> **Exploring Autodesk Ribbon columns**
>
> The user is asking about creating columns in Autodesk's Ribbon UI, specifically in relation to RibbonRowPanel. To achieve multiple columns, you add items sequentially to RibbonPanelSource.Items. Each row can have items, and a RibbonRowBreak is used to start a new column. The IsTopJustified property can help with the layout as well.
>
> **Clarifying the user's request**
>
> The user wants to create a panel with three columns, but the term "flyout" actually refers to the slide-out content, not to a regular column. In Autodesk, columns are created by adding RibbonRowPanels, and a RibbonPanelSource manages the SlideOut property for extra content when needed. The flyout is separate, not part of the column.
>
> **Adding a third column**
>
> To add a third column, I would need to add another RibbonRowPanel with four small buttons, possibly arranged in two rows. Columns are vertical stacks, and inside a column, I can stack rows. A RibbonRowPanel represents a column, and RibbonRowBreak might be used to separate rows. I'd need to confirm this with more specifics.
>
> **Searching Autodesk ribbon API for column creation code**
>
> [RibbonRowBreak Class](https://help.autodesk.com/view/OARX/2024/ENU/?guid=OARX-ManagedRefGuide-Autodesk_Windows_RibbonRowBreak)
> [Ribbon API Slide-Out](https://forums.autodesk.com/t5/net-forum/ribbon-api-slide-out/td-p/3761158)
> [Room Editor Live and Unofficial Custom Ribbon Button](https://thebuildingcoder.typepad.com/blog/2014/06/room-editor-live-and-unofficial-custom-ribbon-button.html)
> [Arrange Ribbon buttons into columns - AutoCAD DevBlog](https://adndevblog.typepad.com/autocad/2012/05/arrange-ribbon-buttons-into-columns.html)
> [Customizing Your Ribbon & Workspace in AutoCAD](https://www.youtube.com/watch?v=YarGZLdOSvw)
> [An Issue With Using Ribbon Runtime API](https://drive-cad-with-code.blogspot.com/2013/12/an-issue-with-using-ribbon-runtime-api.html)
> [AutoCAD CUI Ribbon .NET: Create Ribbon Tab and Panel](https://spiderinnet1.typepad.com/blog/2012/05/autocad-cui-ribbon-net-create-ribbon-tab-and-panel.html)
> [c# - Dynamically add multiple columns in a panel](https://stackoverflow.com/questions/48098775/dynamically-add-multiple-columns-in-a-panel)
> [The New RibbonBar API in AutoCAD 2009](https://through-the-interface.typepad.com/through_the_interface/2008/04/the-new-ribbonb.html)
> [Create a Custom Floating Ribbon Panel for Your Most ...](https://www.landfx.com/kb/autocad-fxcad/customization/5462-floating-ribbon-panel.html)
>
> Thought for 21s

Great q-quick correction first: the “fly-out” is **not** column 2. It’s the panel’s **slide-out** area (shown when you click the tiny arrow). Columns are created by how you add items to the **main** panel; slide-out is a separate bucket.

Here’s how to get exactly what you want:

- **Columns:** Add **one `RibbonRowPanel` per column** to `RibbonPanelSource.Items`.  
- **Rows inside a column:** Insert **`RibbonRowBreak`** inside a `RibbonRowPanel` to start a new row. ([Autodesk Help](https://help.autodesk.com/view/OARX/2024/ENU/?guid=OARX-ManagedRefGuide-Autodesk_Windows_RibbonRowPanel&utm_source=chatgpt.com))  
- **Slide-out:** If you *do* want a fly-out, put a **`RibbonPanelBreak`** in `RibbonPanelSource.Items`; everything **after** that goes to the slide-out. (Or use `RibbonPanelSource.SlideOut`.) ([Autodesk Help](https://help.autodesk.com/view/OARX/2024/ENU/?guid=OARX-ManagedRefGuide-Autodesk_Windows_RibbonPanelBreak&utm_source=chatgpt.com))

### Pattern: add a 3rd column with 4 buttons (2×2 grid)
```csharp
// Assume: var src = new RibbonPanelSource { Title = "Home" };

// Column 1 (whatever you already have)
var col1 = new RibbonRowPanel();
col1.Items.Add(MakeBtn("A", "CMD_A"));
src.Items.Add(col1);

// Column 2 (your existing stuff)
var col2 = new RibbonRowPanel();
col2.Items.Add(MakeBtn("B", "CMD_B"));
src.Items.Add(col2);

// --- Column 3: four buttons arranged 2 per row ---
var col3 = new RibbonRowPanel();
col3.Items.Add(MakeBtn("One",   "CMD_1"));   // row 1
col3.Items.Add(MakeBtn("Two",   "CMD_2"));
col3.Items.Add(new Autodesk.Windows.RibbonRowBreak()); // start row 2
col3.Items.Add(MakeBtn("Three", "CMD_3"));   // row 2
col3.Items.Add(MakeBtn("Four",  "CMD_4"));
src.Items.Add(col3);

// Optional: everything after this break goes to the slide-out (fly-out)
src.Items.Add(new Autodesk.Windows.RibbonPanelBreak());
// ...Add slide-out items here (won’t be in any “column”)
```

Helper (same as we’ve used before):
```csharp
private static RibbonButton MakeBtn(string text, string param) =>
    new RibbonButton {
        Text = text, ShowText = true,
        Size = RibbonItemSize.Standard,
        CommandParameter = param,
        CommandHandler = new RibbonCommandHandler()
    };
```

#### Why this works
- Each **`RibbonRowPanel` == one visible column** on the panel.  
- **`RibbonRowBreak`** splits rows within *that* column, giving you the 2×2 layout. Autodesk’s docs explicitly call out using RowBreak to make multiple rows, and `RibbonRowPanel` as the sub-panel/column. ([Autodesk Help](https://help.autodesk.com/view/OARX/2024/ENU/?guid=OARX-ManagedRefGuide-Autodesk_Windows_RibbonRowPanel&utm_source=chatgpt.com))  
- The **fly-out/slide-out** is controlled by `RibbonPanelBreak` (or `SlideOut`), not by columns. ([Autodesk Help](https://help.autodesk.com/view/OARX/2024/ENU/?guid=OARX-ManagedRefGuide-Autodesk_Windows_RibbonPanelBreak&utm_source=chatgpt.com))

If you want, I can drop this right into your `RibbonInitializer` and wire those four buttons to your real commands (e.g., `opencbisearch`, `openfindform`, etc.).

---

**Sources:**

- [RibbonRowPanel Class](https://help.autodesk.com/view/OARX/2024/ENU/?guid=OARX-ManagedRefGuide-Autodesk_Windows_RibbonRowPanel&utm_source=chatgpt.com)

## Prompt:
9/23/2025, 6:18:05 PM

Yes please

## Response:
9/23/2025, 6:18:07 PM

> **Providing C# code snippet**
>
> I need to give a C# code snippet to modify RibbonInitializer, matching the user's requirements. The code will integrate column 3 (2x2 grid) with a slide-out and show how to add four buttons for commands: `opencbisearch`, `openfindform`, `openopcform`, and `opentagmgr` into the `BuildHomePanel` method. The user prefers specific version and class annotations. I'll focus on providing the snippet directly without asking for clarification.
>
> Thought for 9s

Got it-here’s a drop-in update that adds a **3rd column** to your **Home** panel with a **2×2 grid** of buttons (CBI Search, Find, OPC, TagMgr). It uses `RibbonRowPanel` (column) + `RibbonRowBreak` (new row). I kept your handler mappings (`CBISEARCH`, `FINDFORM`, `OPCFORM`, `TAGMGR`) so it plugs right in.

# RibbonInitializer.cs (target: C# 7.3, LangVersion ≥ 9.0)

```csharp
// File: RibbonInitializer.cs
// Target: AutoCAD 2020
// Language: C# 7.3 (LangVersion 9.0 or greater)
// Version: 1.1.0
// Changelog:
//  - 1.1.0: Added Column 3 to Home panel with a 2x2 grid (CBI Search, Find, OPC, TagMgr)
//  - 1.0.0: Initial "kitchen-sink" ribbon (tab, panels, split/toggle/textbox, slide-out)

using System.Linq;
using Autodesk.AutoCAD.Runtime;
using Autodesk.AutoCAD.ApplicationServices; // per your preference
using Autodesk.Windows;

[assembly: CommandClass(typeof(AutoCAD2020.RibbonInitializer))]

namespace AutoCAD2020
{
    public class RibbonInitializer : IExtensionApplication
    {
        private const string TabId = "CBI_TOOLS_TAB";

        public void Initialize()
        {
            EnsureRibbonVisible();

            var ribbon = ComponentManager.Ribbon;
            if (ribbon == null) return;

            // Create/reuse tab
            var tab = ribbon.Tabs.FirstOrDefault(t => t.Id == TabId)
                      ?? new RibbonTab { Id = TabId, Title = "CBI Tools" };
            if (!ribbon.Tabs.Contains(tab)) ribbon.Tabs.Add(tab);

            // Build Home panel (with new Column 3)
            var home = BuildHomePanel();
            var existing = tab.Panels.FirstOrDefault(p => p.Source?.Title == "Home");
            if (existing != null) tab.Panels.Remove(existing);
            tab.Panels.Add(home);

            tab.IsActive = true;
        }

        public void Terminate() { /* optional cleanup */ }

        private static RibbonPanel BuildHomePanel()
        {
            var src = new RibbonPanelSource { Title = "Home" };

            // -------------------------
            // Column 1 (example content)
            // -------------------------
            var col1 = new RibbonRowPanel();
            col1.Items.Add(new RibbonButton
            {
                Text = "Primary",
                ShowText = true,
                Size = RibbonItemSize.Large,
                CommandParameter = "CBISEARCH",
                CommandHandler = new RibbonCommandHandler(),
                ToolTip = new RibbonToolTip { Title = "CBI Search", Content = "Open CBI Search (opencbisearch)." }
            });
            src.Items.Add(col1);

            // -------------------------
            // Column 2 (example content)
            // -------------------------
            var col2 = new RibbonRowPanel();
            var split = new RibbonSplitButton
            {
                Text = "Run",
                ShowText = true,
                Size = RibbonItemSize.Large,
                IsSplit = true
            };
            split.Items.Add(MakeBtn("Find",  "FINDFORM"));
            split.Items.Add(MakeBtn("OPC",   "OPCFORM"));
            col2.Items.Add(split);
            src.Items.Add(col2);

            // ---------------------------------------------------
            // Column 3: 2x2 grid of buttons (your requested part)
            // ---------------------------------------------------
            var col3 = new RibbonRowPanel();

            // Row 1
            col3.Items.Add(MakeBtn("CBI Search", "CBISEARCH"));
            col3.Items.Add(MakeBtn("Find",       "FINDFORM"));

            // New row inside the same column
            col3.Items.Add(new RibbonRowBreak());

            // Row 2
            col3.Items.Add(MakeBtn("OPC",    "OPCFORM"));
            col3.Items.Add(MakeBtn("TagMgr", "TAGMGR"));

            src.Items.Add(col3);

            // (Optional) Everything after this goes to the slide-out/fly-out panel
            // src.Items.Add(new RibbonPanelBreak());
            // src.Items.Add(MakeBtn("Extra", "EXTRA_CMD"));

            return new RibbonPanel { Source = src };
        }

        private static RibbonButton MakeBtn(string text, string param) =>
            new RibbonButton
            {
                Text = text,
                ShowText = true,
                Size = RibbonItemSize.Standard, // Standard = small square good for grids
                CommandParameter = param,
                CommandHandler = new RibbonCommandHandler()
            };

        private static void EnsureRibbonVisible()
        {
            if (ComponentManager.Ribbon == null)
            {
                var doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
                doc?.SendStringToExecute("RIBBON ", true, false, false);
            }
        }
    }

    public class RibbonCommandHandler : System.Windows.Input.ICommand
    {
        public bool CanExecute(object parameter) => true;
        public event System.EventHandler CanExecuteChanged { add { } remove { } }

        public void Execute(object parameter)
        {
            var item = parameter as RibbonItem;
            var action = (item?.CommandParameter ?? string.Empty).ToString().Trim();
            if (string.IsNullOrEmpty(action)) return;

            var doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;

            switch (action.ToUpperInvariant())
            {
                case "CBISEARCH": doc.SendStringToExecute("opencbisearch ", true, false, false); break;
                case "FINDFORM":  doc.SendStringToExecute("openfindform ", true, false, false);  break;
                case "OPCFORM":   doc.SendStringToExecute("openopcform ",  true, false, false);  break;
                case "TAGMGR":    doc.SendStringToExecute("opentagmgr ",   true, false, false);  break;

                default:
                    doc?.Editor?.WriteMessage($"\n[CBI Ribbon] Unknown action: {action}");
                    break;
            }
        }
    }
}
```

### Notes
- **Why this creates “Column 3”:** Each `RibbonRowPanel` you add to `RibbonPanelSource.Items` becomes a **visible column**. Inside it, `RibbonRowBreak` starts a new **row**-so you get a 2×2 grid in that single column.
- **Button size:** `RibbonItemSize.Standard` gives compact squares that tile nicely. Change to `Large` for tall tiles (vertical label).
- **Icons:** Assign `Image` (16×16) / `LargeImage` (32×32) on each `RibbonButton` later when you’re ready.

If you want these four to live under your **“Tags”** grouping instead of “Home,” I can duplicate the pattern in your Tags panel method too.

## Prompt:
11/3/2025, 3:44:57 PM

“Show me the current workflow schema or architecture summary for this tool.”

## Response:
11/3/2025, 3:44:59 PM

> **Thinking longer for a better answer**
>
> 
>
> Thought for 7s

Here’s the concise **workflow schema / architecture summary** for the **CBI Ribbon** tool (runtime ribbon, no XML/CUIX).

# Purpose
Create and manage a dynamic **“CBI Tools”** ribbon tab in AutoCAD 2020 that routes UI clicks to your existing commands (per your **Command Registration Standard**).

# User-Facing Workflow
1. AutoCAD starts → add-in loads → ribbon appears (`CBI Tools` tab).
2. User clicks buttons/split/toggle/textbox on panels (Home, Utilities, etc.).
3. Each click maps to a short action string → executes your real command (e.g., `opencbisearch`, `openfindform`).

# Runtime Flow
1. **IExtensionApplication.Initialize()**
   - Ensure ribbon UI is visible (`RIBBON`).
   - Create (or reuse) tab `CBI_TOOLS_TAB`.
   - Build panels with columns (`RibbonRowPanel`) and rows (`RibbonRowBreak`).
   - Optionally attach slide-out content (`RibbonPanelBreak` or `SlideOut`).
2. **Interaction**
   - Each control has `CommandParameter` (e.g., `CBISEARCH`).
   - Central **`RibbonCommandHandler`** switches on the parameter and calls `SendStringToExecute(...)` with the actual AutoCAD command.
3. **Terminate()**
   - (Optional) Remove the tab when the DLL unloads.

# Code Structure (C# 7.3, LangVersion ≥ 9.0)
- `AutoCAD2020\RibbonInitializer.cs`
  - `RibbonInitializer : IExtensionApplication`
    - `Initialize()`, `Terminate()`
    - `BuildHomePanel()`, `BuildUtilitiesPanel()`, etc.
    - Helpers: `EnsureRibbonVisible()`, `MakeBtn(...)`
  - `RibbonCommandHandler : ICommand`
    - Routes `CommandParameter` → `SendStringToExecute(...)`
- **No XML/CUIX**; pure runtime API via `Autodesk.Windows` (Ribbon).

# Command Registry Integration (your standard)
- Keep **all** `[CommandMethod]` entries in `AutoCAD2020\Commands.cs` (switchboard only).
- Ribbon buttons call the **real** command names you already expose (e.g., `opencbisearch`), maintaining the one-line routing pattern.

# Ribbon Layout Schema (key concepts)
- **Tab**: `RibbonTab` (“CBI Tools”)
- **Panels**: `RibbonPanel` with `RibbonPanelSource` (e.g., “Home”, “Utilities”)
- **Columns**: each **`RibbonRowPanel`** added to `Source.Items` becomes a **column**
- **Rows in a column**: insert **`RibbonRowBreak`**
- **Slide-out/fly-out**: everything after **`RibbonPanelBreak`** (or use `Source.SlideOut`)
- **Items**: `RibbonButton`, `RibbonSplitButton`, `RibbonToggleButton`, `RibbonTextBox`, `RibbonCombo`, `RibbonSeparator`
- **Sizing**: `RibbonItemSize.Large|Standard`; no absolute width control (influence via columns/rows)

# State & Behavior
- **Toggles**: handled in `RibbonCommandHandler` (set a static flag or invoke a toggle command).
- **TextBox**: `EnterPressed` → send command with the textbox text.
- **Icons**: assign `Image` (16×16) / `LargeImage` (32×32 PNG).

# Error Handling
- Catch and log exceptions in `Initialize()` (write to editor).
- Guard against `ComponentManager.Ribbon == null` (auto-show ribbon).

# Versioning & Docs (your preference)
- File headers with **Version + Changelog**.
- XML docs on public helpers.
- Keep **semantic version** for the ribbon module (e.g., `1.1.0` added Column 3 grid).

# Deployment & Autoload
- Build for AutoCAD 2020 (.NET 4.7.2 typical).
- Place DLL + resources under **`C:\CBICAD\AutoCAD`** (your convention).
- Load via **`NETLOAD`** or autoload (e.g., Registry/Bundle). Ensure references: `acmgd.dll`, `acdbmgd.dll`, `AdWindows.dll`.

# Extensibility Points
- Add more panels (e.g., **Tags**, **QA/QC**, **Sheet Sets**).
- Swap `SendStringToExecute` with direct `.NET` calls if you want tighter coupling.
- Add contextual tabs later (show/hide by selection).

# Dependencies
- `Autodesk.AutoCAD.ApplicationServices`
- `Autodesk.AutoCAD.Runtime`
- `Autodesk.Windows` (AdWindows.dll)

# Known Constraints
- **No fixed panel width** API; layout is driven by columns/rows/item sizes.
- Ribbon is recreated each load; persistent user re-arrangements are not stored unless you implement them separately.

If you want, I can generate a **mini class map** of your current ribbon (tab → panels → columns → rows → items) from the code we just wrote so you can paste it into your Obsidian architecture index.



---
Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)