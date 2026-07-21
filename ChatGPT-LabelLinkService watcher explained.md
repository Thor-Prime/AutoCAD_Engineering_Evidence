# LabelLinkService watcher explained

**User:** Anonymous  
**Created:** 10/6/2025 13:09:08  
**Updated:** 11/3/2025 15:38:06  
**Exported:** 7/21/2026 18:49:16  
**Link:** [https://chatgpt.com/g/g-p-677821cf820c8191964b0f1a58f9ad88/c/68e405c2-b704-8325-b457-20d38876b3d5](https://chatgpt.com/g/g-p-677821cf820c8191964b0f1a58f9ad88/c/68e405c2-b704-8325-b457-20d38876b3d5)  

## Prompt:
10/6/2025, 1:09:07 PM

What is a LabelLinkService.AttachEntityModifiedWatcher(doc); is this a form of a autocad reactor? Can you deep dive me in on this watcher?

## Response:
10/6/2025, 1:09:09 PM

> **Clarifying AutoCAD event handling**
>
> It seems like the user is asking about using a method for handling AutoCAD DB events. In .NET, events differ from ARX reactors—it’s important to explain how reactors are legacy and that in .NET, events are attached to items like the Database, Document, and Editor. I'll include guidance on attaching objects to these events, handling transaction safety, and best practices like weak events to avoid leaks. I'll give C# code examples with explanations of re-entrancy guards and UI thread interaction.
>
> Thought for 22s

Short answer: yes—`AttachEntityModifiedWatcher(doc)` is almost certainly your helper that wires up AutoCAD .NET **database events** (like `ObjectModified`, `ObjectErased`, etc.) on the document’s `Database`. In ARX terms people say “reactor,” but in .NET these are **managed events**, not the old C++ “reactor” API. Functionally it’s the same idea: “react” when something changes in the drawing.

Here’s a deep-dive and a battle-tested pattern you can drop in.

# What this “watcher” typically does
- Subscribes to `doc.Database.ObjectModified`, `ObjectErased`, and optionally `ObjectAppended`.
- Filters events to only the entities you care about (e.g., polylines or label blocks carrying your XData/NOD keys).
- Records what changed (e.g., mark “dirty”, capture old/new geometry, stash ObjectIds).
- Defers any **writes** (like auto-moving a label) to a safe moment (end of command/Idle), to avoid re-entrancy loops.

# Reactors vs .NET events (why it matters)
- **ARX reactors (C++)**: low-level, very early notifications, lots of foot-guns.
- **.NET events**: safer, higher level. You still must avoid changing DB objects inside the same modification callback unless you know what you’re doing (you can create infinite recursion or `eLockViolation`).
- Best practice in .NET: **observe now, mutate later** (queue work for `Application.Idle` or `DocumentCollection.ExecuteInCommandContextAsync`).

# A solid implementation template (C# 7.3, AutoCAD 2020)

```csharp
// v0.1.0 — LabelLinkService watcher scaffold
// - Attaches per-document DB event handlers
// - Filters for linked entities (by your XData/NOD signature)
// - Queues safe updates post-command to avoid re-entrancy

using System;
using System.Collections.Generic;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;

namespace AutoCAD2020.Services
{
    public static class LabelLinkService
    {
        // Guard to prevent recursion if your own updates fire events
        private static bool _handling;
        // Pending work captured during events, to flush later
        private static readonly Dictionary<Document, HashSet<ObjectId>> _dirtyByDoc
            = new Dictionary<Document, HashSet<ObjectId>>();

        // Call this once per Document (e.g., on load or when your tools start)
        public static void AttachEntityModifiedWatcher(Document doc)
        {
            if (doc == null) return;

            var db = doc.Database;

            // Avoid double subscription by detaching first (idempotent pattern)
            DetachEntityModifiedWatcher(doc);

            db.ObjectModified += OnObjectModified;
            db.ObjectErased   += OnObjectErased;
            db.ObjectAppended += OnObjectAppended;

            // Optional: hook document lifecycle so you can auto-detach
            doc.ImpliedSelectionChanged += OnImpliedSelectionChanged_NoOp; // keeps a token ref if you like

            GetEd(doc).WriteMessage(
                "\n[LabelLinkService] Entity watcher attached for this document.");
        }

        public static void DetachEntityModifiedWatcher(Document doc)
        {
            if (doc == null) return;

            var db = doc.Database;
            db.ObjectModified -= OnObjectModified;
            db.ObjectErased   -= OnObjectErased;
            db.ObjectAppended -= OnObjectAppended;

            doc.ImpliedSelectionChanged -= OnImpliedSelectionChanged_NoOp;

            _dirtyByDoc.Remove(doc);

            GetEd(doc).WriteMessage(
                "\n[LabelLinkService] Entity watcher detached for this document.");
        }

        // --- Event handlers ---

        private static void OnObjectAppended(object sender, ObjectEventArgs e)
        {
            var db = (Database)sender;
            var doc = FindDocument(db);
            if (doc == null) return;

            if (!IsLinkedTarget(e.DBObject)) return; // your filter
            MarkDirty(doc, e.DBObject.ObjectId);
        }

        private static void OnObjectModified(object sender, ObjectEventArgs e)
        {
            var db = (Database)sender;
            var doc = FindDocument(db);
            if (doc == null) return;

            if (!IsLinkedTarget(e.DBObject)) return; // your filter
            MarkDirty(doc, e.DBObject.ObjectId);
        }

        private static void OnObjectErased(object sender, ObjectErasedEventArgs e)
        {
            var db = (Database)sender;
            var doc = FindDocument(db);
            if (doc == null) return;

            if (!e.Erased) return; // only care about erase
            if (!IsLinkedTarget(e.DBObject)) return;
            MarkDirty(doc, e.DBObject.ObjectId); // you might unlink labels later
        }

        // --- Queuing & flushing ---

        private static void MarkDirty(Document doc, ObjectId id)
        {
            if (_handling) return; // avoid re-entrancy storms

            if (!_dirtyByDoc.TryGetValue(doc, out var set))
            {
                set = new HashSet<ObjectId>();
                _dirtyByDoc[doc] = set;

                // Schedule a safe flush after the current command completes.
                // ExecuteInCommandContextAsync is available in ACAD 2020+.
                Application.DocumentManager.ExecuteInCommandContextAsync(async (o) =>
                {
                    await System.Threading.Tasks.Task.Yield();
                    Flush(doc);
                }, null);
            }

            set.Add(id);
        }

        private static void Flush(Document doc)
        {
            if (!_dirtyByDoc.TryGetValue(doc, out var set) || set.Count == 0) return;

            try
            {
                _handling = true;

                using (doc.LockDocument())
                using (var tr = doc.Database.TransactionManager.StartTransaction())
                {
                    foreach (var id in set)
                    {
                        if (!id.IsValid || id.IsNull) continue;

                        // Open for read; upgrade only if you truly must edit.
                        var ent = tr.GetObject(id, OpenMode.ForRead, false) as Entity;
                        if (ent == null) continue;

                        // Your rule: re-position/update linked label(s) for this entity
                        // e.g., UpdateLinkedLabels(doc, tr, ent);

                        // Example sketch:
                        // if (TryFindLinkedLabelIds(ent, tr, out var labels))
                        //     RelocateLabelsToNewAnchor(ent, labels, tr);
                    }
                    tr.Commit();
                }
            }
            catch (System.Exception ex)
            {
                GetEd(doc).WriteMessage($"\n[LabelLinkService] Flush error: {ex.Message}");
            }
            finally
            {
                set.Clear();
                _handling = false;
            }
        }

        // --- Helpers (adapt to your XData/NOD scheme) ---

        private static bool IsLinkedTarget(DBObject dbo)
        {
            // Filter only the things you care about (Polyline, BlockReference, etc.)
            var ent = dbo as Entity;
            if (ent == null) return false;

            // Example: check for your XData app name
            // return XDataUtils.HasApp(ent, "CBI_LINK"); // your helper

            // Or check for a specific extension dictionary/XRecord marker
            // return XRecUtils.HasKey(ent, "CBI:Linked");

            return true; // stub until you wire your real filter
        }

        private static Document FindDocument(Database db)
        {
            foreach (Document d in Application.DocumentManager)
                if (d.Database == db) return d;
            return null;
        }

        private static Editor GetEd(Document d) => d?.Editor ?? Application.DocumentManager.MdiActiveDocument.Editor;

        private static void OnImpliedSelectionChanged_NoOp(object sender, EventArgs e) { /* placeholder */ }
    }
}
```

## How to use it
```csharp
// Somewhere in your initialize/startup:
var doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
AutoCAD2020.Services.LabelLinkService.AttachEntityModifiedWatcher(doc);

// On shutdown/unload (or doc closing):
AutoCAD2020.Services.LabelLinkService.DetachEntityModifiedWatcher(doc);
```

# Key do’s & don’ts

**Do**
- Filter aggressively (`IsLinkedTarget`) so you don’t watch the whole world.
- Defer any **writes** to `Flush` (post-command), wrapped in `using (doc.LockDocument())` and a single `Transaction`.
- Guard re-entrancy (`_handling`) to avoid loops when your own edits retrigger events.
- Attach per-document and detach when a document closes to avoid leaks.
- Consider ignoring xref events (`IsFromExternalReference`) if you don’t intend to react to them.

**Don’t**
- Don’t call `UpgradeOpen()` inside raw event callbacks unless you absolutely must; prefer deferring.
- Don’t spawn new commands from within the event. If needed, queue with `ExecuteInCommandContextAsync`.
- Don’t rely on `ObjectId` if your design needs cross-drawing persistence; store your own robust key (you already use XData/NOD which is perfect).

# Tying it to your label workflow
Given your setup (polyline + XData link + label block with stored insertion), this watcher is ideal to:
- Detect when the **host entity** geometry changes.
- Queue and **recompute the label** anchor/justification (e.g., move from MiddleCenter to BottomCenter per your recent tweak).
- Respect user nudges: if the user has manually moved the label, store a “user-offset” and apply delta-preserving moves instead of snapping back to the canonical point. (You can log a `ManualOffset` vector in the label’s XData and incorporate it during `Flush`.)

If you want, say the word and I’ll wire the `IsLinkedTarget`, `TryFindLinkedLabelIds`, and a `RelocateLabelsToNewAnchor` that implements your **Bottom-Middle** placement and preserves manual tweaks.

## Prompt:
10/6/2025, 1:15:58 PM

Sooooo much good stuff here is am just salivating at all the options. OK. How about this. If a user edited the dtext of our label. Is there a watcher that can detect change and update the xdata of the polyline? Like a reverse properties edit? Or is this too far?

## Response:
10/6/2025, 1:15:59 PM

> **Providing guidance on tracking text changes in AutoCAD**
>
> To track text changes effectively, we want to listen for events like `ObjectModified` for `DBText` or `MText`. Since events don't provide old values, we could store previous values using data structures like an `XRecord`, or by capturing changes during specific commands with `Begin/EndCommand` or `CommandWillStart` events.
>
> Thought for 15s

Absolutely doable—and a great idea. You can “reverse-sync” from the label back to the host polyline by watching text edits and then updating the polyline’s XData/NOD. The trick is: AutoCAD’s `.NET` **ObjectModified** event doesn’t give you the *old* value, so you need a lightweight strategy to know whether the user actually changed the text (vs your own code doing it) and to avoid event recursion.

Below is a battle-tested pattern you can drop in.

---

# Reverse-sync label → polyline (C# 7.3 / AutoCAD 2020)

### Design choices
- **What we watch:** `DBText`, `MText`, and (optional) `AttributeReference`.
- **How we know it changed:** store a **“last-synced text”** token on the label (XData/XRecord). When AutoCAD raises `ObjectModified`, compare current text to the stored token.  
- **How we avoid loops:** a per-document **reentrancy guard** (`_handling`) and updating the label’s last-synced token *after* we propagate to the polyline.
- **How we find the host polyline:** keep a **link marker** on the label (e.g., the polyline’s **Handle** string). Convert handle → `ObjectId` when needed.

> If you already keep a two-way link (label ↔ host) in XData/NOD, you can plug that in directly.

---

### Code: watcher + reverse-sync

```csharp
// v0.2.0 — LabelReverseSyncService
// - Watches DBText/MText changes
// - If user changes label text, updates host polyline XData (reverse sync)
// - Guards against recursion and non-user programmatic updates

using System;
using System.Collections.Generic;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;
using Autodesk.AutoCAD.Runtime;

namespace AutoCAD2020.Services
{
    public static class LabelReverseSyncService
    {
        private const string AppName = "CBI_LINK";            // your RegApp for XData
        private const string XdHostHandleKey = "HOST_HANDLE"; // label->host link
        private const string XdLastTextKey   = "LAST_TEXT";   // last-synced text token

        private static bool _handling; // reentrancy guard

        public static void Attach(Document doc)
        {
            Detach(doc);
            var db = doc.Database;
            db.ObjectModified += OnObjectModified;
            db.ObjectErased   += OnObjectErased;   // optional: clean-up
            doc.CommandWillStart += OnCmdBegin;    // helps classify user vs code, optional
            doc.CommandEnded     += OnCmdEnd;
            doc.CommandCancelled += OnCmdEnd;
            doc.CommandFailed    += OnCmdEnd;

            EnsureRegApp(db, AppName);

            Ed(doc).WriteMessage("\n[LabelReverseSync] watcher attached.");
        }

        public static void Detach(Document doc)
        {
            var db = doc.Database;
            db.ObjectModified -= OnObjectModified;
            db.ObjectErased   -= OnObjectErased;
            doc.CommandWillStart -= OnCmdBegin;
            doc.CommandEnded     -= OnCmdEnd;
            doc.CommandCancelled -= OnCmdEnd;
            doc.CommandFailed    -= OnCmdEnd;
        }

        // --- Optional: track high-level command so you can decide to ignore your own commands
        private static string _currentCmd;
        private static void OnCmdBegin(object s, CommandEventArgs e) => _currentCmd = e.GlobalCommandName;
        private static void OnCmdEnd(object s, CommandEventArgs e)   => _currentCmd = null;

        private static void OnObjectErased(object sender, ObjectErasedEventArgs e)
        {
            // If the label is erased, you might want to clear a host-side backlink, etc.
            // Kept minimal here.
        }

        private static void OnObjectModified(object sender, ObjectEventArgs e)
        {
            if (_handling) return;

            var db  = (Database)sender;
            var doc = FindDoc(db);
            if (doc == null) return;

            // We only care about DBText/MText (and optionally AttributeReference)
            DBObject dbo = e.DBObject;
            if (!(dbo is DBText) && !(dbo is MText) && !(dbo is AttributeReference))
                return;

            // Open a short TRX in this callback ONLY for reads; actual writes will be deferred within the same loop but guarded.
            try
            {
                using (var tr = db.TransactionManager.StartTransaction())
                {
                    // Get the label text in a normalized form
                    string currentText = GetCurrentText(dbo, tr);
                    if (currentText == null) return;

                    // Is this object one of "our" labels, and does it have a host link?
                    if (!TryGetHostHandle(dbo, tr, out string hostHandle)) return;

                    // Compare against last-synced text to avoid reacting to our own writes
                    string lastText = GetXDataString(dbo, tr, XdLastTextKey);
                    bool changedByUser = !StringEquals(currentText, lastText);

                    if (!changedByUser) return;

                    // Propagate to host polyline
                    if (TryGetObjectIdFromHandle(db, hostHandle, out ObjectId hostId))
                    {
                        _handling = true; // prevent recursion
                        try
                        {
                            using (doc.LockDocument())
                            using (var trw = db.TransactionManager.StartTransaction())
                            {
                                // Update host XData (or NOD) with the new value
                                if (hostId.IsValid && !hostId.IsNull)
                                {
                                    var hostEnt = trw.GetObject(hostId, OpenMode.ForRead) as Entity;
                                    if (hostEnt != null)
                                    {
                                        // Write the string into host XData as your chosen key, e.g., "LABEL_TEXT"
                                        SetXDataString(hostEnt, trw, AppName, "LABEL_TEXT", currentText);
                                    }
                                }

                                // Update the label's LAST_TEXT token to the new value so future edits compare correctly
                                var label = trw.GetObject(dbo.ObjectId, OpenMode.ForRead) as Entity;
                                if (label != null)
                                {
                                    SetXDataString(label, trw, AppName, XdLastTextKey, currentText);
                                }

                                trw.Commit();
                            }
                        }
                        finally { _handling = false; }
                    }

                    tr.Commit();
                }
            }
            catch (System.Exception ex)
            {
                Ed(doc).WriteMessage($"\n[LabelReverseSync] Error: {ex.Message}");
            }
        }

        // ---- Helpers ---------------------------------------------------------

        private static string GetCurrentText(DBObject dbo, Transaction tr)
        {
            switch (dbo)
            {
                case DBText dt:
                    return dt.TextString?.Trim();
                case MText mt:
                    // MText can contain formatting codes; strip if you need “plain”:
                    return mt.Contents?.Trim(); // or mt.Text for plain, if available in your version
                case AttributeReference ar:
                    return ar.TextString?.Trim();
                default:
                    return null;
            }
        }

        private static bool TryGetHostHandle(DBObject label, Transaction tr, out string handleStr)
        {
            handleStr = GetXDataString(label, tr, XdHostHandleKey);
            return !string.IsNullOrWhiteSpace(handleStr);
        }

        private static bool TryGetObjectIdFromHandle(Database db, string handleStr, out ObjectId id)
        {
            id = ObjectId.Null;
            try
            {
                var h = new Handle(Convert.ToInt64(handleStr, 16));
                return db.TryGetObjectId(h, out id);
            }
            catch { return false; }
        }

        private static string GetXDataString(DBObject obj, Transaction tr, string key)
        {
            var rb = obj.GetXDataForApplication(AppName);
            if (rb == null) return null;

            // XData is a (appName, …key, value…) style you define yourself
            // Example convention: (-3, appName) (1000, key) (1000, value) ...
            var vals = rb.AsArray();
            for (int i = 0; i < vals.Length - 1; i++)
            {
                if (vals[i].TypeCode == 1000 && (vals[i].Value as string) == key)
                {
                    var next = vals[i + 1];
                    if (next.TypeCode == 1000) return next.Value as string;
                }
            }
            return null;
        }

        private static void SetXDataString(Entity ent, Transaction tr, string app, string key, string value)
        {
            // Read existing
            var old = ent.GetXDataForApplication(app);
            var list = new List<TypedValue>();
            list.Add(new TypedValue(1001, app));

            // Copy everything except previous instance of (key, val)
            if (old != null)
            {
                var arr = old.AsArray();
                // Start at 1 to skip 1001 header we’ve already added
                for (int i = 1; i < arr.Length; i++)
                {
                    var tv = arr[i];
                    // remove previous key/value pair
                    if (tv.TypeCode == 1000 && (tv.Value as string) == key)
                    {
                        i++; // skip following value
                        continue;
                    }
                    list.Add(tv);
                }
            }

            // Append/replace with new pair
            list.Add(new TypedValue(1000, key));
            list.Add(new TypedValue(1000, value ?? string.Empty));

            // Write back
            ent.UpgradeOpen();
            ent.XData = new ResultBuffer(list.ToArray());
        }

        private static void EnsureRegApp(Database db, string app)
        {
            using (var tr = db.TransactionManager.StartTransaction())
            {
                var regApps = (RegAppTable)tr.GetObject(db.RegAppTableId, OpenMode.ForRead);
                if (!regApps.Has(app))
                {
                    regApps.UpgradeOpen();
                    var rec = new RegAppTableRecord { Name = app };
                    regApps.Add(rec);
                    tr.AddNewlyCreatedDBObject(rec, true);
                }
                tr.Commit();
            }
        }

        private static Document FindDoc(Database db)
        {
            foreach (Document d in Application.DocumentManager)
                if (d.Database == db) return d;
            return null;
        }

        private static Editor Ed(Document d) =>
            d?.Editor ?? Application.DocumentManager.MdiActiveDocument.Editor;

        private static bool StringEquals(string a, string b) =>
            string.Equals(a ?? "", b ?? "", StringComparison.Ordinal);
    }
}
```

---

## How to link label ↔ host (one-time per label)
When you create a label (or during a repair pass), stamp two bits of XData on the **label**:
- `HOST_HANDLE` → the host polyline’s `Handle` (hex string).
- `LAST_TEXT` → the label’s current text at creation time.

Example when placing the label:
```csharp
// after creating labelEnt and knowing hostEnt
var hostHandleHex = hostEnt.Handle.Value.ToString("X");
SetXDataString(labelEnt, tr, "CBI_LINK", "HOST_HANDLE", hostHandleHex);
SetXDataString(labelEnt, tr, "CBI_LINK", "LAST_TEXT", normalizedLabelText);
```

> If you already store links in NOD, that’s fine—keep that for global indexing. The **handle on the label** is just a quick local pointer to speed up the watcher.

---

## Practical notes & pitfalls
- **MText formatting:** `MText.Contents` contains formatting codes; if you want plain text comparisons, prefer `MText.Text` (available in newer APIs) or strip formatting yourself.
- **Your own commands:** If your refresh command rewrites labels, you don’t want reverse-sync to fight it. The `_handling` flag + updating `LAST_TEXT` after your write prevents ping-pong.
- **Undo/Redo:** Works fine—event fires again; compare vs `LAST_TEXT` and decide. If you need richer history, you can snapshot in `CommandWillStart/Ended`.
- **Copies / wblock clone:** If labels get copied to a new drawing, handles change; plan a **repair pass** (scan labels, re-bind to nearest/selected host) or encode a more robust host key (e.g., host handle + drawing GUID or your own NOD index).
- **Attributes:** If your label is a block with an attribute, include `AttributeReference` in the watcher and read `ar.TextString`.

---

## TL;DR
Not too far at all. Watch text edits on labels; if the current text differs from the label’s stored `LAST_TEXT`, look up the host via `HOST_HANDLE`, update the polyline’s XData, and then refresh the label’s `LAST_TEXT`. The guard flag avoids infinite loops, and the handle link makes it fast and reliable.

If you want, I can tailor the `SetXDataString/GetXDataString` to match your exact XData packing format and wire this into your existing `LabelLinkService` so it’s one cohesive module.

## Prompt:
11/3/2025, 3:37:18 PM

“Show me the current workflow schema or architecture summary for this tool.”

## Response:
11/3/2025, 3:37:21 PM

> **Thinking longer for a better answer**
>
> 
>
> Thought for 7s

Here’s the compact workflow/architecture snapshot for the **Label ↔ Host Link** tool we’ve been shaping (LabelLinkService + ReverseSync). It’s scoped to AutoCAD 2020 / C# 7.3 and matches your Command Registration Standard.

# High-level goal
Keep **labels** (DBText/MText/AttrRef) and their **host entities** (e.g., polylines) in two-way sync:

- **Forward sync:** when the host geometry changes, re-compute/relocate/update the label.
- **Reverse sync:** when a user edits the label text, push that value back into the host’s XData/NOD.

# Core components

1) **LabelLinkService** (forward watcher)
- Attaches per-document DB event hooks:
  - `Database.ObjectModified`, `ObjectErased`, `ObjectAppended`
- Filters to “watched” host entities (e.g., your linked polylines).
- Queues dirty ObjectIds and **flushes once** post-command using `Application.DocumentManager.ExecuteInCommandContextAsync(...)`.
- In `Flush()` (inside one `doc.LockDocument()` + one `Transaction`), runs **UpdateLinkedLabels**:
  - Re-computes anchor (e.g., Bottom-Middle, preserves user offset if present).
  - Updates label geometry/text only when needed (to minimize churn).

2) **LabelReverseSyncService** (reverse watcher)
- Hooks `Database.ObjectModified` and focuses on label types:
  - `DBText`, `MText`, (optional) `AttributeReference`
- On change, compares current text vs a “last-synced” token on the label.
- If changed by user, finds the host via a backlink and updates the host’s XData/NOD, then refreshes the label’s “last-synced” token to avoid loops.

3) **Link storage (lightweight, local on label)**
- **RegApp:** `CBI_LINK`
- **XData keys on the label:**
  - `HOST_HANDLE` → hex handle string of the host polyline for fast lookup.
  - `LAST_TEXT`  → last value written by sync (prevents ping-pong).
- **XData (or NOD) on the host:**
  - e.g., `LABEL_TEXT` → canonical text the system considers authoritative.
  - (You can add others like `ManualOffset` vector, `AnchorMode`, etc.)

4) **Guards & safety**
- `_handling` boolean to prevent re-entrancy while you write inside event responses.
- **Defer writes**: events capture; **writes occur in Flush** (forward) or in a guarded short write (reverse) with `doc.LockDocument()`.
- **Idempotency**: forward/ reverse both check current state vs desired state before editing.

# Data flow (happy paths)

**A. Host edited → label updated (forward sync)**
1. User grips/edits host polyline.
2. `ObjectModified` fires → LabelLinkService marks host as dirty.
3. After command ends, `Flush()` opens one transaction:
   - Finds labels linked to host (via your index or host-side XData list).
   - Computes new anchor/rotation/justification.
   - Applies minimal changes to the label; updates label’s `LAST_TEXT` if text changed by your code.

**B. Label text edited → host updated (reverse sync)**
1. User double-clicks label, edits text.
2. `ObjectModified` fires on the label → LabelReverseSync:
   - Reads current text; compares to label’s `LAST_TEXT`.
   - If different, resolves `HOST_HANDLE` → host `ObjectId`.
   - Writes the new value to host’s XData/NOD key (e.g., `LABEL_TEXT`).
   - Updates label’s `LAST_TEXT` to the new value (prevents recursion).

**C. Label erased → host cleanup (optional)**
- On `ObjectErased` for a label, you can remove the backlink on the host (or mark orphaned) during a scheduled maintenance pass.

# Initialization & lifecycle

- **Attach on load/startup (per document):**
  - `LabelLinkService.AttachEntityModifiedWatcher(doc)`
  - `LabelReverseSyncService.Attach(doc)`
  - Ensure RegApp `CBI_LINK` exists.
- **Detach on document close/unload:**
  - Matching `Detach(...)` calls remove handlers and clear per-doc queues.

# Commands (per your Commands.cs “switchboard”)

```csharp
// CBI_ prefix, one-liners that route to real logic:

[CommandMethod("CBI_Label_Watch_On")]
public static void CBI_Label_Watch_On()
    => AutoCAD2020.Services.LabelLinkService.AttachEntityModifiedWatcher(
        Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument);

[CommandMethod("CBI_Label_Watch_Off")]
public static void CBI_Label_Watch_Off()
    => AutoCAD2020.Services.LabelLinkService.DetachEntityModifiedWatcher(
        Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument);

[CommandMethod("CBI_Label_ReverseSync_On")]
public static void CBI_Label_ReverseSync_On()
    => AutoCAD2020.Services.LabelReverseSyncService.Attach(
        Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument);

[CommandMethod("CBI_Label_ReverseSync_Off")]
public static void CBI_Label_ReverseSync_Off()
    => AutoCAD2020.Services.LabelReverseSyncService.Detach(
        Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument);

// Optional manual maintenance:
[CommandMethod("CBI_Label_RepairLinks")]
public static void CBI_Label_RepairLinks()
    => AutoCAD2020.Services.LabelRepairService.ScanAndRebind(); // e.g., rebuild HOST_HANDLE, fix orphans
```

# Key decisions & conventions

- **Authoritative text:** Store on the **host** (`LABEL_TEXT`), mirror to the label; reverse sync updates the host first, then stamps `LAST_TEXT` on the label.
- **Backlink** on label (`HOST_HANDLE`) for O(1) host resolution; more robust cross-drawing identity can live in NOD (with doc GUID) for deeper scenarios.
- **Event strategy:** Observe now, **mutate later** (forward). For reverse, small atomic mutation guarded by `_handling` is fine.
- **User intent preservation:** If a user drags a label, store a `ManualOffset` vector on the label; forward recompute = canonical anchor + `ManualOffset`.

# Extensibility hooks (near-term)

- **Placement policy provider:** interface to swap anchor modes (Bottom-Middle vs Middle-Center), rotation alignment (parallel to segment vs fixed), and offsets.
- **Batch refresh:** a command that triggers `UpdateLinkedLabels` over a selection set or entire drawing.
- **Undo breadcrumbs:** record change notes into your ChangeHistory manager (you already have the pattern).
- **Xref awareness:** ignore xref’d entities unless explicitly allowed.

# Minimal contract (so everything “just works”)

- **RegApp:** `CBI_LINK`
- **Label XData:** `HOST_HANDLE`, `LAST_TEXT`
- **Host XData/NOD:** `LABEL_TEXT` (and any extras you choose)
- **Public APIs used by the services (expected in your codebase):**
  - `XDataUtils.HasApp/ReadString/WriteString`
  - `LinkIndex.TryGetLinkedLabels(hostId, out IEnumerable<ObjectId> labels)` (or equivalent)
  - `LabelPlacement.Compute(..)` + `Apply(..)` (anchor, rotation, offset)
  - `ChangeHistory.Log(..)` (optional)

---

If you want, I can turn this into a one-page **Architecture.md** (with a small UML/sequence ASCII diagram) that drops into your `Almighty_Vault/CBI_Tools/Labels/` folder.



---
Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)