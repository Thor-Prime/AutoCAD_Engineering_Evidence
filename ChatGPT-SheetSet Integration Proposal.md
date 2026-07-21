# SheetSet Integration Proposal

**User:** Anonymous  
**Created:** 12/31/2025 12:49:52  
**Updated:** 3/24/2026 15:34:54  
**Exported:** 7/21/2026 18:57:51  
**Link:** [https://chatgpt.com/g/g-p-677821cf820c8191964b0f1a58f9ad88/c/6955704e-bd50-8328-ae51-2af72f52d808](https://chatgpt.com/g/g-p-677821cf820c8191964b0f1a58f9ad88/c/6955704e-bd50-8328-ae51-2af72f52d808)  

## Prompt:
12/31/2025, 12:59:17 PM

Architect’s Lens ON.  
  
Load context from prior work:  
- SOP: CBI_ProjectStore_SOP_v1.0  
- Project_Store.json schema: v1.0 (full template)  
- UX Contracts:  
  - Attach Sheet Set UX Contract v1.0  
  - Sync UX Contract v1.0  
- Decisions locked:  
  - ProjectState conflict handling: Synced-Degraded  
  - Drawing Status model: Active / MissingFile / Archived  
  - Core logic in AutoCAD_Shared_Project (Option A)  
  - AutoCAD references only in AutoCAD2020 project  
  - No dialogs in normal tools  
  - NOD canonical key: CBI.ProjectStore.Path  
  - Legacy key migration without illegal DBDictionary access  
  
Active Tool:  
- ProjectStore Sheet Set Integration  
  (Attach Sheet Set + Sync Sheet Set)  
  
Scope Guard (MANDATORY):  
- Allowed:  
  - AutoCAD_Shared_Project.ProjectStore (core sync logic)  
  - AutoCAD2020.ProjectStore (commands + AutoCAD SheetSet access)  
  - New helper types required for sheet traversal, sync, conflicts  
- Not allowed:  
  - OPC tools implementation (architecture only later)  
  - Tag allocation changes beyond read-only usage  
  - Valve tools or tagging commands  
  - UI beyond command-line Editor output  
  
Objective for this thread:  
- Implement code for:  
  1) SheetSet attach command  
  2) SheetSet sync command  
  3) Core sync service (no AutoCAD refs)  
  4) Conflict detection + persistence  
  5) ProjectState + Health transitions  
- Produce drop-in, versioned C# code (C# 7.3 compatible)  
- Follow SOP v1.0 exactly; no schema drift  
  
Begin with:  
- Class list and file layout proposal  
- Then proceed to drop-in code  
  
If code is required to update existing classes please ask for the existing class code to review and then suggest changes.

Solution Architecture Context (Reference Only):

- Visual Studio Solution:
  - AutoCAD_Shared_Project
    - ProjectStore
    - NOD_X-Data
    - Shared utilities (no AutoCAD refs)

  - AutoCAD2020
    - ProjectStore (commands, SheetSet access)
    - Commands.cs (central command registry)
    - Uses AutoCAD 2020 references

- Standards:
  - Commands registered only in AutoCAD2020.Commands.cs
  - Shared project must not reference AutoCAD assemblies
  - All ProjectStore schema changes are versioned
  - Drop-in files must respect existing namespaces

## Response:
12/31/2025, 12:59:18 PM

> **Planning file layout and class structure**
>
> I need to provide a proposal for the file and class layout first, per the user’s request. I'll include the namespaces: `AutoCAD_Shared_Project.ProjectStore` and `AutoCAD2020.ProjectStore`. I'll also mention the commands in `Commands.cs` for the attach and sync functions, but won’t provide code just yet—just a brief outline with notes for now.
>
> **Proceeding with class list and code**
>
> The user asked for a class list to start and then for drop-in code, so I’ll provide both in the same response. I’ll include new classes like `IProjectStoreRepository` to wrap around existing ones, which should help avoid direct dependency calls. I'll focus on adapting and isolating, and ensure compatibility with `ProjectStoreIo` and `Validator`.
>
> **Implementing attach command and validations**
>
> For the attach command, I need to set the NOD key `CBI.ProjectStore.Path` to the store file path, which might need to reference the sheet set’s destination path (`ProjectStore.Config.SheetSet.Path`). I’ll use `Editor.GetString` for the destination input, ensuring it accepts spaces and validates the file path, since dialogs aren't allowed.
>
> Thought for 18s

## Scope Guard Check

Active Tool remains **ProjectStore Sheet Set Integration** (Attach + Sync). The plan and code below stay within the allowed areas:
- **AutoCAD_Shared_Project.ProjectStore**: core sync logic, conflict detection, state transitions (no AutoCAD refs)
- **AutoCAD2020.ProjectStore**: commands + Sheet Set Manager traversal (AutoCAD refs only)
- No dialogs beyond **Editor command-line prompts**; no tagging/valve allocation changes; no OPC tools.

---

## Class list and file layout proposal

### AutoCAD_Shared_Project (no AutoCAD references)

**Folder:** `AutoCAD_Shared_Project/ProjectStore/SheetSetIntegration/`

1) `SheetSetSnapshot.cs`  
- Pure DTOs that describe a sheet set at a point in time (path, sheets, subsets, resolved DWG paths).

2) `SheetSetSyncRequest.cs`  
- Input contract for sync: store model (loaded), snapshot, policy flags.

3) `SheetSetSyncResult.cs`  
- Output contract: changes applied, conflicts found, state/health transition results, warnings.

4) `ProjectStateMachine.cs`  
- Implements **ProjectState + Health transitions** (including **Synced-Degraded** conflict handling).

5) `SheetSetSyncService.cs`  
- Core service: compare snapshot vs store, detect conflicts, produce updates + persistence payload (no I/O).

6) `ConflictModels.cs`  
- Conflict types and persistence-friendly representations.

7) `PathNormalization.cs`  
- Canonicalization helpers for drawing paths and comparisons.

> Note: This layer assumes the existing shared types exist: `ProjectStoreModel` (schema v1.0), `ProjectStoreValidator`, `ProjectStoreIo` (atomic save), and store “History/Conflicts/Health/ProjectState” fields per your SOP v1.0. If any of those names/signatures differ, **we will not edit them without reviewing your current class code**—instead we will adjust the adapter calls in AutoCAD2020.

---

### AutoCAD2020 (AutoCAD references allowed)

**Folder:** `AutoCAD2020/ProjectStore/SheetSetIntegration/`

1) `SheetSetTraversal.cs`  
- Reads a `.dst` using Sheet Set Manager APIs and produces a shared `SheetSetSnapshot`.

2) `ProjectStoreResolver.cs`  
- Reads **NOD canonical key** `CBI.ProjectStore.Path` safely and returns the store file path.  
- Contains the “legacy key migration” hook **without illegal DBDictionary access** (implemented as “attempt read new key; if missing, attempt read legacy key via allowed API path; if found, write new key via allowed API path”).

3) `Command_AttachSheetSet.cs`  
- Implements attach UX contract v1.0 via Editor prompts.

4) `Command_SyncSheetSet.cs`  
- Implements sync UX contract v1.0 via Editor prompts and result reporting.

5) `Commands.cs` update  
- Adds `[CommandMethod]` switchboard entries only (no logic).

---

## Drop-in, versioned C# code (C# 7.3 compatible)

Below are new files you can drop in as-is. Where I *must* integrate with your existing store model/IO/validator, I’ve done it behind small interfaces so you can wire it up without modifying your established code. If you want me to directly call your existing `ProjectStoreIo` / `ProjectStoreValidator` / store model types, paste those class files and I will refit the adapters precisely.

---

### 1) AutoCAD_Shared_Project — DTOs and core sync logic

#### File: `AutoCAD_Shared_Project/ProjectStore/SheetSetIntegration/SheetSetSnapshot.cs`
```csharp
// -----------------------------------------------------------------------------
// Project: AutoCAD_Shared_Project
// File: SheetSetSnapshot.cs
// Version: 1.0.0
// Changelog:
//   1.0.0 - Initial DTOs for Sheet Set snapshot (no AutoCAD refs).
// -----------------------------------------------------------------------------

using System;
using System.Collections.Generic;

namespace AutoCAD_Shared_Project.ProjectStore.SheetSetIntegration
{
    [Serializable]
    public sealed class SheetSetSnapshot
    {
        public string SheetSetPath { get; set; }          // Full path to .dst
        public string SheetSetName { get; set; }          // Friendly name (if available)
        public DateTime CapturedUtc { get; set; }         // Snapshot capture time
        public List<SheetSnapshot> Sheets { get; set; } = new List<SheetSnapshot>();
        public List<SubsetSnapshot> Subsets { get; set; } = new List<SubsetSnapshot>();

        // Convenience: flattened index by "SheetId" (stable within snapshot)
        public Dictionary<string, SheetSnapshot> SheetById { get; set; } = new Dictionary<string, SheetSnapshot>(StringComparer.OrdinalIgnoreCase);
    }

    [Serializable]
    public sealed class SubsetSnapshot
    {
        public string SubsetId { get; set; }
        public string Name { get; set; }
        public string ParentSubsetId { get; set; } // null for root
    }

    [Serializable]
    public sealed class SheetSnapshot
    {
        public string SheetId { get; set; }               // SSM object id (string form)
        public string Name { get; set; }                  // Sheet name
        public string Number { get; set; }                // Sheet number
        public string SubsetId { get; set; }              // Which subset the sheet belongs to
        public string DrawingPath { get; set; }           // Resolved DWG path (may be empty)
        public bool DrawingMissing { get; set; }          // True if path resolves but file not found (if caller checks)
        public Dictionary<string, string> CustomProps { get; set; } = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);
    }
}
```

#### File: `AutoCAD_Shared_Project/ProjectStore/SheetSetIntegration/ConflictModels.cs`
```csharp
// -----------------------------------------------------------------------------
// Project: AutoCAD_Shared_Project
// File: ConflictModels.cs
// Version: 1.0.0
// Changelog:
//   1.0.0 - Initial conflict model set for Synced-Degraded policy.
// -----------------------------------------------------------------------------

using System;
using System.Collections.Generic;

namespace AutoCAD_Shared_Project.ProjectStore.SheetSetIntegration
{
    public enum ProjectConflictType
    {
        Unknown = 0,

        // Sheet set says one thing, store says another
        SheetSetPathMismatch,
        MissingSheetSetFile,

        // Snapshot includes sheet with missing/moved drawing
        MissingDrawingFile,

        // Snapshot includes duplicate drawing path entries (ambiguous)
        DuplicateDrawingReference,

        // Store references drawing that no longer exists in sheet set
        StoreDrawingNotInSheetSet,

        // Any other integrity mismatch
        IntegrityViolation
    }

    [Serializable]
    public sealed class ProjectConflict
    {
        public string ConflictId { get; set; }            // GUID string
        public ProjectConflictType Type { get; set; }
        public string Severity { get; set; }             // "Warning" | "Error" (string to match your schema style)
        public string Message { get; set; }
        public string DrawingPath { get; set; }          // Optional
        public string SheetId { get; set; }              // Optional
        public DateTime CreatedUtc { get; set; }
        public Dictionary<string, string> Meta { get; set; } = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase);

        public static ProjectConflict Create(ProjectConflictType type, string severity, string message)
        {
            return new ProjectConflict
            {
                ConflictId = Guid.NewGuid().ToString("D"),
                Type = type,
                Severity = severity ?? "Warning",
                Message = message ?? string.Empty,
                CreatedUtc = DateTime.UtcNow
            };
        }
    }
}
```

#### File: `AutoCAD_Shared_Project/ProjectStore/SheetSetIntegration/PathNormalization.cs`
```csharp
// -----------------------------------------------------------------------------
// Project: AutoCAD_Shared_Project
// File: PathNormalization.cs
// Version: 1.0.0
// Changelog:
//   1.0.0 - Path normalization utilities for stable comparisons.
// -----------------------------------------------------------------------------

using System;
using System.IO;

namespace AutoCAD_Shared_Project.ProjectStore.SheetSetIntegration
{
    public static class PathNormalization
    {
        public static string Normalize(string path)
        {
            if (string.IsNullOrWhiteSpace(path))
                return string.Empty;

            try
            {
                // Normalize separators and casing for comparisons
                var full = Path.GetFullPath(path.Trim());
                return full.Replace(Path.AltDirectorySeparatorChar, Path.DirectorySeparatorChar)
                           .TrimEnd(Path.DirectorySeparatorChar)
                           .ToUpperInvariant();
            }
            catch
            {
                // If it is not a valid path, return best-effort normalized
                return path.Trim()
                           .Replace(Path.AltDirectorySeparatorChar, Path.DirectorySeparatorChar)
                           .TrimEnd(Path.DirectorySeparatorChar)
                           .ToUpperInvariant();
            }
        }

        public static bool EqualsNormalized(string a, string b)
            => string.Equals(Normalize(a), Normalize(b), StringComparison.OrdinalIgnoreCase);
    }
}
```

#### File: `AutoCAD_Shared_Project/ProjectStore/SheetSetIntegration/SheetSetSyncRequest.cs`
```csharp
// -----------------------------------------------------------------------------
// Project: AutoCAD_Shared_Project
// File: SheetSetSyncRequest.cs
// Version: 1.0.0
// Changelog:
//   1.0.0 - Sync request contract.
// -----------------------------------------------------------------------------

namespace AutoCAD_Shared_Project.ProjectStore.SheetSetIntegration
{
    public sealed class SheetSetSyncRequest<TProjectStore>
    {
        public TProjectStore Store { get; set; }
        public SheetSetSnapshot Snapshot { get; set; }

        // Policies locked by SOP v1.0
        public string ConflictHandlingPolicy { get; set; } = "Synced-Degraded";
        public bool AllowStoreUpdate { get; set; } = true;   // still no dialogs; just policy gate
    }
}
```

#### File: `AutoCAD_Shared_Project/ProjectStore/SheetSetIntegration/SheetSetSyncResult.cs`
```csharp
// -----------------------------------------------------------------------------
// Project: AutoCAD_Shared_Project
// File: SheetSetSyncResult.cs
// Version: 1.0.0
// Changelog:
//   1.0.0 - Sync result contract.
// -----------------------------------------------------------------------------

using System;
using System.Collections.Generic;

namespace AutoCAD_Shared_Project.ProjectStore.SheetSetIntegration
{
    public sealed class SheetSetSyncResult
    {
        public bool Success { get; set; }
        public string Summary { get; set; }

        // ProjectState + Health transitions
        public string PreviousProjectState { get; set; }
        public string NewProjectState { get; set; }
        public string PreviousHealth { get; set; }
        public string NewHealth { get; set; }

        public DateTime CompletedUtc { get; set; }

        public List<string> AppliedChanges { get; } = new List<string>();
        public List<ProjectConflict> Conflicts { get; } = new List<ProjectConflict>();
        public List<string> Warnings { get; } = new List<string>();
    }
}
```

#### File: `AutoCAD_Shared_Project/ProjectStore/SheetSetIntegration/ProjectStateMachine.cs`
```csharp
// -----------------------------------------------------------------------------
// Project: AutoCAD_Shared_Project
// File: ProjectStateMachine.cs
// Version: 1.0.0
// Changelog:
//   1.0.0 - Implements state/health transitions with Synced-Degraded policy.
// -----------------------------------------------------------------------------

using System.Collections.Generic;

namespace AutoCAD_Shared_Project.ProjectStore.SheetSetIntegration
{
    public static class ProjectStateMachine
    {
        // Locked decisions:
        // - ProjectState conflict handling: Synced-Degraded
        // - Drawing Status model: Active / MissingFile / Archived
        public static void ApplySyncedDegradedPolicy(
            int conflictErrorCount,
            int conflictWarningCount,
            out string newProjectState,
            out string newHealth)
        {
            // Conservative policy:
            // - Any "Error" conflict => Synced-Degraded (Health: Degraded)
            // - Only warnings => Synced (Health: Warning) OR Degraded depending on SOP preference
            // Here we implement:
            //   Errors => ProjectState="Synced-Degraded", Health="Degraded"
            //   Warnings only => ProjectState="Synced", Health="Warning"
            //   None => ProjectState="Synced", Health="Healthy"

            if (conflictErrorCount > 0)
            {
                newProjectState = "Synced-Degraded";
                newHealth = "Degraded";
                return;
            }

            if (conflictWarningCount > 0)
            {
                newProjectState = "Synced";
                newHealth = "Warning";
                return;
            }

            newProjectState = "Synced";
            newHealth = "Healthy";
        }

        public static int CountSeverity(IReadOnlyList<ProjectConflict> conflicts, string severity)
        {
            if (conflicts == null || conflicts.Count == 0)
                return 0;

            var count = 0;
            for (int i = 0; i < conflicts.Count; i++)
            {
                if ((conflicts[i]?.Severity ?? string.Empty) == severity)
                    count++;
            }
            return count;
        }
    }
}
```

#### File: `AutoCAD_Shared_Project/ProjectStore/SheetSetIntegration/SheetSetSyncService.cs`
```csharp
// -----------------------------------------------------------------------------
// Project: AutoCAD_Shared_Project
// File: SheetSetSyncService.cs
// Version: 1.0.0
// Changelog:
//   1.0.0 - Core sync logic (no AutoCAD refs). Detects conflicts and drives
//           state/health transitions. Store persistence is performed by caller.
// -----------------------------------------------------------------------------

using System;
using System.Collections.Generic;
using System.IO;

namespace AutoCAD_Shared_Project.ProjectStore.SheetSetIntegration
{
    /// <summary>
    /// Core service: compares SheetSetSnapshot with the ProjectStore model and produces
    /// conflicts + state transitions. This class performs no file IO on the store itself.
    /// </summary>
    public sealed class SheetSetSyncService
    {
        public SheetSetSyncResult Sync<TProjectStore>(
            SheetSetSyncRequest<TProjectStore> request,
            IProjectStoreSheetSetBinding<TProjectStore> binding)
        {
            if (request == null) throw new ArgumentNullException(nameof(request));
            if (binding == null) throw new ArgumentNullException(nameof(binding));

            var result = new SheetSetSyncResult
            {
                Success = false,
                CompletedUtc = DateTime.UtcNow
            };

            if (request.Store == null)
            {
                result.Summary = "Store is null; cannot sync.";
                return result;
            }

            if (request.Snapshot == null)
            {
                result.Summary = "Sheet set snapshot is null; cannot sync.";
                return result;
            }

            // Capture prior state
            result.PreviousProjectState = binding.GetProjectState(request.Store);
            result.PreviousHealth = binding.GetHealth(request.Store);

            // 1) Validate sheet set path binding
            var storeSheetSetPath = binding.GetBoundSheetSetPath(request.Store);
            var snapshotSheetSetPath = request.Snapshot.SheetSetPath;

            if (!string.IsNullOrWhiteSpace(storeSheetSetPath) &&
                !PathNormalization.EqualsNormalized(storeSheetSetPath, snapshotSheetSetPath))
            {
                var c = ProjectConflict.Create(
                    ProjectConflictType.SheetSetPathMismatch,
                    "Error",
                    "Bound SheetSet path in ProjectStore does not match the snapshot SheetSet path.");
                c.Meta["StoreSheetSetPath"] = storeSheetSetPath ?? "";
                c.Meta["SnapshotSheetSetPath"] = snapshotSheetSetPath ?? "";
                result.Conflicts.Add(c);
            }

            // 2) Missing sheet set file (caller can choose to validate; we do it here best-effort)
            if (!string.IsNullOrWhiteSpace(snapshotSheetSetPath) && !File.Exists(snapshotSheetSetPath))
            {
                var c = ProjectConflict.Create(
                    ProjectConflictType.MissingSheetSetFile,
                    "Error",
                    "SheetSet (.dst) file does not exist at the snapshot path.");
                c.Meta["SheetSetPath"] = snapshotSheetSetPath ?? "";
                result.Conflicts.Add(c);
            }

            // 3) Index drawing paths, detect duplicates
            var drawingPathToSheetIds = new Dictionary<string, List<string>>(StringComparer.OrdinalIgnoreCase);

            for (int i = 0; i < request.Snapshot.Sheets.Count; i++)
            {
                var s = request.Snapshot.Sheets[i];
                var norm = PathNormalization.Normalize(s?.DrawingPath);

                if (string.IsNullOrWhiteSpace(norm))
                    continue;

                if (!drawingPathToSheetIds.TryGetValue(norm, out var list))
                {
                    list = new List<string>();
                    drawingPathToSheetIds[norm] = list;
                }
                list.Add(s.SheetId ?? string.Empty);

                // Missing drawing
                // We treat missing as Warning by default; SOP can elevate if needed.
                if (s.DrawingMissing)
                {
                    var c = ProjectConflict.Create(
                        ProjectConflictType.MissingDrawingFile,
                        "Warning",
                        "A sheet references a drawing file that is missing.");
                    c.DrawingPath = s.DrawingPath;
                    c.SheetId = s.SheetId;
                    c.Meta["SheetName"] = s.Name ?? "";
                    c.Meta["SheetNumber"] = s.Number ?? "";
                    result.Conflicts.Add(c);
                }
            }

            foreach (var kvp in drawingPathToSheetIds)
            {
                if (kvp.Value.Count > 1)
                {
                    var c = ProjectConflict.Create(
                        ProjectConflictType.DuplicateDrawingReference,
                        "Error",
                        "Multiple sheets reference the same drawing path (ambiguous).");
                    c.DrawingPath = kvp.Key;
                    c.Meta["SheetIds"] = string.Join(",", kvp.Value);
                    result.Conflicts.Add(c);
                }
            }

            // 4) Store vs sheet set membership checks (read-only usage beyond tag allocation)
            // NOTE: We do not assume your internal schema; binding provides enumeration.
            var storeDrawingPaths = binding.EnumerateStoreDrawingPaths(request.Store);
            if (storeDrawingPaths != null)
            {
                foreach (var storePath in storeDrawingPaths)
                {
                    var normStore = PathNormalization.Normalize(storePath);
                    if (string.IsNullOrWhiteSpace(normStore))
                        continue;

                    if (!drawingPathToSheetIds.ContainsKey(normStore))
                    {
                        var c = ProjectConflict.Create(
                            ProjectConflictType.StoreDrawingNotInSheetSet,
                            "Warning",
                            "ProjectStore references a drawing that is not present in the current SheetSet snapshot.");
                        c.DrawingPath = storePath;
                        result.Conflicts.Add(c);
                    }
                }
            }

            // 5) Apply state/health transitions per locked policy
            var errorCount = ProjectStateMachine.CountSeverity(result.Conflicts, "Error");
            var warningCount = ProjectStateMachine.CountSeverity(result.Conflicts, "Warning");

            ProjectStateMachine.ApplySyncedDegradedPolicy(
                errorCount,
                warningCount,
                out var newProjectState,
                out var newHealth);

            result.NewProjectState = newProjectState;
            result.NewHealth = newHealth;

            // 6) Persist conflicts + update store binding if allowed
            if (request.AllowStoreUpdate)
            {
                binding.SetProjectState(request.Store, result.NewProjectState);
                binding.SetHealth(request.Store, result.NewHealth);
                binding.AppendConflicts(request.Store, result.Conflicts);
                binding.TouchLastSyncUtc(request.Store, DateTime.UtcNow);

                // Optional: if store has no bound sheet set path, bind it now
                if (string.IsNullOrWhiteSpace(storeSheetSetPath) && !string.IsNullOrWhiteSpace(snapshotSheetSetPath))
                {
                    binding.SetBoundSheetSetPath(request.Store, snapshotSheetSetPath);
                    result.AppliedChanges.Add("Bound SheetSet path was set in the ProjectStore.");
                }
            }

            result.Success = true;
            result.Summary = $"Sync complete. Conflicts: {result.Conflicts.Count}. State={result.NewProjectState}, Health={result.NewHealth}.";
            return result;
        }
    }

    /// <summary>
    /// Adapter interface to connect sync logic to your ProjectStore schema v1.0 without changing shared core code.
    /// Implement this in AutoCAD_Shared_Project.ProjectStore (or AutoCAD2020 as a bridge) once we confirm your model.
    /// </summary>
    public interface IProjectStoreSheetSetBinding<TProjectStore>
    {
        string GetBoundSheetSetPath(TProjectStore store);
        void SetBoundSheetSetPath(TProjectStore store, string sheetSetPath);

        string GetProjectState(TProjectStore store);
        void SetProjectState(TProjectStore store, string projectState);

        string GetHealth(TProjectStore store);
        void SetHealth(TProjectStore store, string health);

        IEnumerable<string> EnumerateStoreDrawingPaths(TProjectStore store);

        void AppendConflicts(TProjectStore store, IEnumerable<ProjectConflict> conflicts);
        void TouchLastSyncUtc(TProjectStore store, DateTime utcNow);
    }
}
```

---

### 2) AutoCAD2020 — Commands + Sheet Set traversal (AutoCAD refs only)

> These files intentionally keep **all AutoCAD and SheetSetManager references** in AutoCAD2020.

#### File: `AutoCAD2020/ProjectStore/SheetSetIntegration/ProjectStoreResolver.cs`
```csharp
// -----------------------------------------------------------------------------
// Project: AutoCAD2020
// File: ProjectStoreResolver.cs
// Version: 1.0.0
// Changelog:
//   1.0.0 - Resolve ProjectStore path from NOD canonical key CBI.ProjectStore.Path.
//           Includes legacy key fallback hook (no illegal DBDictionary access).
// -----------------------------------------------------------------------------

using System;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;

namespace AutoCAD2020.ProjectStore.SheetSetIntegration
{
    internal static class ProjectStoreResolver
    {
        // Locked: NOD canonical key
        public const string CanonicalNodKey = "CBI.ProjectStore.Path";

        // If you have a legacy key, put it here (example only).
        // IMPORTANT: Migration must avoid illegal DBDictionary access. Use only allowed APIs
        // you already vetted in SOP v1.0. This placeholder assumes you already have a safe reader.
        public const string LegacyNodKey = "CBI/ProjectStore/Path";

        public static bool TryGetProjectStorePath(out string storePath, out string message)
        {
            storePath = null;
            message = null;

            var doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
            var ed = doc.Editor;

            try
            {
                using (doc.LockDocument())
                {
                    var db = doc.Database;

                    // Canonical read path: attempt canonical key first
                    if (TryReadNodString(db, CanonicalNodKey, out storePath))
                    {
                        if (!string.IsNullOrWhiteSpace(storePath))
                            return true;
                    }

                    // Legacy fallback (hook)
                    if (TryReadNodString(db, LegacyNodKey, out var legacy))
                    {
                        if (!string.IsNullOrWhiteSpace(legacy))
                        {
                            // Safe migration: write canonical key using allowed method
                            // (still using same safe writer helper here; replace if your SOP mandates a different safe path)
                            TryWriteNodString(db, CanonicalNodKey, legacy);
                            storePath = legacy;
                            message = "Legacy NOD key migrated to canonical key.";
                            return true;
                        }
                    }
                }

                message = $"ProjectStore path not found in NOD. Expected key: {CanonicalNodKey}";
                return false;
            }
            catch (System.Exception ex)
            {
                message = "Failed to resolve ProjectStore path from NOD: " + ex.Message;
                ed.WriteMessage("\n" + message);
                return false;
            }
        }

        // NOTE: These helpers are intentionally minimal.
        // If your SOP v1.0 prescribes a specific NOD helper class, replace calls here
        // after you paste your existing implementation (per your rule).
        private static bool TryReadNodString(Database db, string key, out string value)
        {
            value = null;
            try
            {
                using (var tr = db.TransactionManager.StartTransaction())
                {
                    var nod = (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId, OpenMode.ForRead);
                    if (!nod.Contains(key))
                        return false;

                    var xrecId = nod.GetAt(key);
                    var xrec = (Xrecord)tr.GetObject(xrecId, OpenMode.ForRead);
                    if (xrec?.Data == null)
                        return false;

                    foreach (TypedValue tv in xrec.Data)
                    {
                        if (tv.TypeCode == (int)DxfCode.Text || tv.TypeCode == (int)DxfCode.ExtendedDataAsciiString)
                        {
                            value = tv.Value as string;
                            break;
                        }
                    }

                    tr.Commit();
                    return !string.IsNullOrWhiteSpace(value);
                }
            }
            catch
            {
                return false;
            }
        }

        private static bool TryWriteNodString(Database db, string key, string value)
        {
            try
            {
                using (var tr = db.TransactionManager.StartTransaction())
                {
                    var nod = (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId, OpenMode.ForWrite);

                    Xrecord xrec;
                    if (nod.Contains(key))
                    {
                        xrec = (Xrecord)tr.GetObject(nod.GetAt(key), OpenMode.ForWrite);
                    }
                    else
                    {
                        xrec = new Xrecord();
                        nod.SetAt(key, xrec);
                        tr.AddNewlyCreatedDBObject(xrec, true);
                    }

                    xrec.Data = new ResultBuffer(new TypedValue((int)DxfCode.Text, value ?? string.Empty));
                    tr.Commit();
                    return true;
                }
            }
            catch
            {
                return false;
            }
        }
    }
}
```

> **Important note (Scope Guard / SOP):** Your prompt explicitly says “legacy key migration without illegal DBDictionary access.” The helper above uses `NamedObjectsDictionaryId` and `DBDictionary`, which you previously flagged as potentially problematic in your SOP context.  
> To comply strictly: **paste your approved NOD read/write helper** (the one you already validated as “legal” in your environment), and I will replace `TryReadNodString/TryWriteNodString` with your safe implementation. I did not assume what you consider “legal,” so I’m stopping short of asserting this meets SOP.

---

#### File: `AutoCAD2020/ProjectStore/SheetSetIntegration/SheetSetTraversal.cs`
```csharp
// -----------------------------------------------------------------------------
// Project: AutoCAD2020
// File: SheetSetTraversal.cs
// Version: 1.0.0
// Changelog:
//   1.0.0 - Traverses a Sheet Set (.dst) and produces a shared SheetSetSnapshot.
// -----------------------------------------------------------------------------

using System;
using System.IO;
using System.Runtime.InteropServices;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.EditorInput;

// Digi-Ants.AutoCAD.SheetSetManager (per your prior work)
using Autodesk.AutoCAD.Interop; // often needed for COM plumbing in some environments
using Autodesk.AutoCAD.Interop.Common;

using AutoCAD_Shared_Project.ProjectStore.SheetSetIntegration;

// If you are using Digi-Ants.AutoCAD.SheetSetManager, your namespaces may differ.
// Replace these using directives with your actual references once you confirm the exact types available:
// using DigiAnts.AutoCAD.SheetSetManager;
// using DigiAnts.AutoCAD.SheetSetManager.AcadSm;

namespace AutoCAD2020.ProjectStore.SheetSetIntegration
{
    internal static class SheetSetTraversal
    {
        /// <summary>
        /// Reads the sheet set at dstPath and produces a snapshot suitable for core sync.
        /// </summary>
        public static bool TryCaptureSnapshot(string dstPath, out SheetSetSnapshot snapshot, out string message)
        {
            snapshot = null;
            message = null;

            if (string.IsNullOrWhiteSpace(dstPath))
            {
                message = "SheetSet path is empty.";
                return false;
            }

            if (!File.Exists(dstPath))
            {
                message = "SheetSet file does not exist: " + dstPath;
                // Still create snapshot with missing file marker so conflicts can record it.
                snapshot = new SheetSetSnapshot
                {
                    SheetSetPath = dstPath,
                    SheetSetName = Path.GetFileNameWithoutExtension(dstPath),
                    CapturedUtc = DateTime.UtcNow
                };
                return true;
            }

            try
            {
                // NOTE:
                // The exact traversal depends on which SheetSetManager wrapper you have:
                // - Autodesk.AcSmSheetSetMgr COM
                // - Digi-Ants wrapper around the COM interfaces
                //
                // Because you locked "core logic in shared" and "AutoCAD refs only in AutoCAD2020",
                // we keep traversal here, but we need your *confirmed* API types to finalize.
                //
                // For now, we provide a safe placeholder that captures the basic header.
                snapshot = new SheetSetSnapshot
                {
                    SheetSetPath = dstPath,
                    SheetSetName = Path.GetFileNameWithoutExtension(dstPath),
                    CapturedUtc = DateTime.UtcNow
                };

                // TODO (requires your confirmed SSM API types):
                // - Load database via AcSmSheetSetMgr.GetDatabase(dstPath, true)
                // - Traverse IAcSmSubset hierarchy
                // - For each IAcSmSheet:
                //     - Extract Name, Number, and drawing path (via layout reference / Callout / file reference)
                // - Populate snapshot.Subsets and snapshot.Sheets
                // - Optionally mark DrawingMissing based on File.Exists for resolved DWG paths

                message = "Snapshot captured (header only). Paste your SSM traversal code/types so we can finalize traversal.";
                return true;
            }
            catch (System.Exception ex)
            {
                message = "Failed to capture SheetSet snapshot: " + ex.Message;
                return false;
            }
        }
    }
}
```

> This traversal file is intentionally “stubbed” at the sheet enumeration layer because your environment varies significantly depending on whether you are using:
- Raw COM (`AcSmSheetSetMgr`, `IAcSmSubset`, `IAcSmSheet`), or
- Digi-Ants wrapper types (namespaces differ)

Per your rule, I will not guess the exact class names/methods and risk schema/UX drift. **Paste the current working snippet you mentioned earlier that traverses full hierarchy using `IAcSmSubset`**, and I will drop it in here exactly.

---

#### File: `AutoCAD2020/ProjectStore/SheetSetIntegration/Command_AttachSheetSet.cs`
```csharp
// -----------------------------------------------------------------------------
// Project: AutoCAD2020
// File: Command_AttachSheetSet.cs
// Version: 1.0.0
// Changelog:
//   1.0.0 - Attach Sheet Set UX Contract v1.0 (Editor prompt only).
// -----------------------------------------------------------------------------

using System;
using System.IO;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.EditorInput;

namespace AutoCAD2020.ProjectStore.SheetSetIntegration
{
    internal static class Command_AttachSheetSet
    {
        public static void Execute()
        {
            var doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
            var ed = doc.Editor;

            // 1) Resolve ProjectStore path from NOD (canonical key)
            if (!ProjectStoreResolver.TryGetProjectStorePath(out var storePath, out var msg))
            {
                ed.WriteMessage("\n[ProjectStore] " + msg);
                ed.WriteMessage("\n[ProjectStore] Attach aborted.");
                return;
            }

            if (!string.IsNullOrWhiteSpace(msg))
                ed.WriteMessage("\n[ProjectStore] " + msg);

            // 2) Ask for .dst path via command line (no dialogs)
            var pso = new PromptStringOptions("\nEnter full path to Sheet Set (.dst): ")
            {
                AllowSpaces = true
            };
            var pr = ed.GetString(pso);
            if (pr.Status != PromptStatus.OK)
            {
                ed.WriteMessage("\n[ProjectStore] Attach canceled.");
                return;
            }

            var dstPath = (pr.StringResult ?? string.Empty).Trim().Trim('"');
            if (string.IsNullOrWhiteSpace(dstPath))
            {
                ed.WriteMessage("\n[ProjectStore] No path provided.");
                return;
            }

            // 3) Basic validation (file may be missing; we still allow binding per SOP if desired)
            if (!dstPath.EndsWith(".dst", StringComparison.OrdinalIgnoreCase))
            {
                ed.WriteMessage("\n[ProjectStore] Path does not look like a .dst file.");
                return;
            }

            ed.WriteMessage($"\n[ProjectStore] Store: {storePath}");
            ed.WriteMessage($"\n[ProjectStore] SheetSet: {dstPath}");

            // 4) Persist binding into store
            // IMPORTANT: Do not modify your existing ProjectStore classes without reviewing them.
            // Provide your ProjectStoreIo/Validator signatures and your store model type,
            // and we will wire the attach into the real schema v1.0 fields.

            ed.WriteMessage("\n[ProjectStore] Attach is staged. Paste your ProjectStore model + IO classes so we can write the SheetSet path into Config per SOP v1.0.");
        }
    }
}
```

---

#### File: `AutoCAD2020/ProjectStore/SheetSetIntegration/Command_SyncSheetSet.cs`
```csharp
// -----------------------------------------------------------------------------
// Project: AutoCAD2020
// File: Command_SyncSheetSet.cs
// Version: 1.0.0
// Changelog:
//   1.0.0 - Sync UX Contract v1.0 (Editor output only), core sync in shared.
// -----------------------------------------------------------------------------

using System;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.EditorInput;

using AutoCAD_Shared_Project.ProjectStore.SheetSetIntegration;

namespace AutoCAD2020.ProjectStore.SheetSetIntegration
{
    internal static class Command_SyncSheetSet
    {
        public static void Execute()
        {
            var doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
            var ed = doc.Editor;

            // 1) Resolve ProjectStore path
            if (!ProjectStoreResolver.TryGetProjectStorePath(out var storePath, out var msg))
            {
                ed.WriteMessage("\n[ProjectStore] " + msg);
                ed.WriteMessage("\n[ProjectStore] Sync aborted.");
                return;
            }

            if (!string.IsNullOrWhiteSpace(msg))
                ed.WriteMessage("\n[ProjectStore] " + msg);

            ed.WriteMessage($"\n[ProjectStore] Store: {storePath}");

            // 2) Load store model (requires your existing ProjectStoreIo + schema v1.0 model)
            // 3) Capture sheet set snapshot from store-bound dst path
            // 4) Run shared sync service
            // 5) Persist store (atomic save) and print results to Editor

            ed.WriteMessage("\n[ProjectStore] Sync staged. Paste your ProjectStore model + ProjectStoreIo + ProjectStoreValidator so we can implement end-to-end load/validate/save per SOP v1.0.");
        }
    }
}
```

---

### 3) AutoCAD2020 — Commands switchboard entries (must be in Commands.cs)

Per your command registration standard, add these in `AutoCAD2020/Commands.cs`:

```csharp
// -----------------------------------------------------------------------------
// Commands.cs additions (switchboard only)
// -----------------------------------------------------------------------------

using Autodesk.AutoCAD.Runtime;
using AutoCAD2020.ProjectStore.SheetSetIntegration;

namespace AutoCAD2020
{
    public partial class Commands
    {
        [CommandMethod("CBI_ProjectStore_AttachSheetSet")]
        public void CBI_ProjectStore_AttachSheetSet()
            => Command_AttachSheetSet.Execute();

        [CommandMethod("CBI_ProjectStore_SyncSheetSet")]
        public void CBI_ProjectStore_SyncSheetSet()
            => Command_SyncSheetSet.Execute();
    }
}
```

---

## What I need from you to complete the end-to-end implementation (required by your rule)

To proceed without violating: “If code is required to update existing classes please ask for the existing class code to review and then suggest changes,” please paste (or upload) the current versions of:

1) **ProjectStore schema v1.0 C# model type(s)**  
- The class that represents the JSON root (often `ProjectStoreModel`), plus any nested types used for:
  - `Version`, `Config`, `Project`, `Systems`, `Blocks.Valve.Types`, `Tags`, `Released`, `History`
  - Your **ProjectState** and **Health** fields (wherever they live)

2) **ProjectStoreIo** (atomic save)  
- Load + Save signatures (and whether it returns a model, throws, etc.)

3) **ProjectStoreValidator**  
- Validate signature and how it reports failures (exceptions vs result object)

4) **Your approved NOD read/write helper** for `CBI.ProjectStore.Path`  
- The implementation that satisfies your “no illegal DBDictionary access” constraint.

5) **Your working Sheet Set traversal snippet** (the one using `IAcSmSubset` full hierarchy), or the exact wrapper library types you are using (Digi-Ants vs raw COM).

Once you provide those, I will:
- Finish `SheetSetTraversal` with full subset/sheet enumeration
- Implement `Attach` to write the `.dst` binding into **Config** exactly per SOP v1.0
- Implement `Sync` to load/validate store, run shared `SheetSetSyncService`, persist conflicts/history/state, and print a deterministic Editor summary
- Ensure **ProjectState conflict handling = Synced-Degraded** and drawing status model is applied as **Active / MissingFile / Archived** (no schema drift)

If you paste those files, I will produce a complete “drop-in” set with a version header + changelog for every touched file.

## Prompt:
12/31/2025, 1:37:15 PM

Would you please provide a file tree of the affected files as well as new files please.

## Response:
12/31/2025, 1:37:17 PM

Below is the proposed **file tree** showing **new files** and **affected (modified) files** for the **ProjectStore Sheet Set Integration** tool only.

---

## AutoCAD_Shared_Project

### New files
```
AutoCAD_Shared_Project
└─ ProjectStore
   └─ SheetSetIntegration
      ├─ SheetSetSnapshot.cs
      ├─ SheetSetSyncRequest.cs
      ├─ SheetSetSyncResult.cs
      ├─ SheetSetSyncService.cs
      ├─ ProjectStateMachine.cs
      ├─ ConflictModels.cs
      └─ PathNormalization.cs
```

### Affected files (modified)
```
AutoCAD_Shared_Project
└─ (none yet)
```

> Note: No shared files are modified at this stage because the sync core is isolated behind `IProjectStoreSheetSetBinding<TProjectStore>`. Once you paste your store model + IO + validator, we may add **one** binding implementation file in Shared (or in AutoCAD2020 as a bridge), but I will not assume that until we review your existing code.

---

## AutoCAD2020

### New files
```
AutoCAD2020
└─ ProjectStore
   └─ SheetSetIntegration
      ├─ Command_AttachSheetSet.cs
      ├─ Command_SyncSheetSet.cs
      ├─ ProjectStoreResolver.cs
      └─ SheetSetTraversal.cs
```

### Affected files (modified)
```
AutoCAD2020
└─ Commands.cs
```

---

## Summary count

- **New files:** 11
  - Shared: 7
  - AutoCAD2020: 4
- **Modified files:** 1
  - AutoCAD2020: `Commands.cs`

If you want, I can also provide a “future expected” tree (after you paste the existing ProjectStore model/IO/validator), showing where the binding adapter will live and which file(s) will be modified—without writing any changes until we review your code.

## Prompt:
12/31/2025, 3:45:41 PM

// ============================================================================
// File      : AutoCAD_Shared_Project/ProjectStore/ProjectStoreV1.cs
// Purpose   : v1 data model for the Project Store (JSON) used by CBI tools.
// Target    : C# 7.3 / .NET Framework compatible
// Notes     : Pure POCOs. No helper logic here — TagStoreV1/StoreGuards live in
//             namespace AutoCAD_Shared_Project.ProjectStore (parent, no ".V1").
// ============================================================================

using System;
using System.Collections.Generic;

namespace AutoCAD_Shared_Project.ProjectStore.V1
{
    public sealed class ProjectInfoV1
    {
        public string Name { get; set; } = "";
    }

    // ------------------------------- Config ----------------------------------
    public sealed class StoreConfigV1
    {
        public string GlobalValvePrefix { get; set; } = "V";
        public int GraceDays { get; set; } = 7;     // days before auto-release allowed
        public bool PreferReleasedLowestFirst { get; set; } = true;  // reuse smallest number first
    }

    // ------------------------------- Counters --------------------------------
    public sealed class SystemCounterV1
    {
        // Example: Prefix="INST", Pad=4 -> INST-0001
        public string Prefix { get; set; } = "INST";
        public int Pad { get; set; } = 4;
        public int Start { get; set; } = 1;
        public int End { get; set; } = 9999;
        public int Next { get; set; } = 1;   // next number to allocate
    }

    // ------------------------------- Status ----------------------------------
    public enum TagStatusV1
    {
        Active = 0,
        Orphan = 1,
        Released = 2,
        Reserved = 3,  // keep for future use / UI filters
        Superseded = 4
    }

    // ------------------------------- History ---------------------------------
    public sealed class TagHistoryV1
    {
        public DateTime Utc { get; set; } = DateTime.UtcNow;
        public string Action { get; set; } = "";   // Allocated, Released, Reassigned, MarkedOrphan, etc.
        public string By { get; set; } = "";
        public string Details { get; set; } = "";
    }

    public sealed class HistoryEntryV1
    {
        public DateTime Utc { get; set; } = DateTime.UtcNow;
        public string Action { get; set; } = "";   // Allocate, AllocateFromReleased, Released, etc.
        public string By { get; set; } = "";
        public string Details { get; set; } = "";
    }

    // ------------------------------- Records ---------------------------------
    public sealed class TagRecordV1
    {
        // Identity
        public string Id { get; set; } = "";        // e.g., "INST-0021"
        public string System { get; set; } = "GENERAL"; // system key used to allocate

        // Placement (lightweight)
        public string Drawing { get; set; } = "";        // file path or name
        public string EntityHandle { get; set; } = "";        // owner block handle (hex)

        // Lifecycle
        public TagStatusV1 Status { get; set; } = TagStatusV1.Active;
        public DateTime CreatedUtc { get; set; } = DateTime.UtcNow;
        public DateTime? OrphanedAtUtc { get; set; } = null;

        // Audit
        public string By { get; set; } = "";
        public string Notes { get; set; } = "";

        // Per-tag history
        public List<TagHistoryV1> History { get; set; } = new List<TagHistoryV1>();
    }

    public sealed class TagReleaseRecordV1
    {
        public string Id { get; set; } = "";        // released tag ID
        public string Reason { get; set; } = "";
        public string By { get; set; } = "";
        public DateTime Utc { get; set; } = DateTime.UtcNow;
    }

    // ------------------------------- Root Store -------------------------------
    public sealed class ProjectStoreV1
    {
        public string Version { get; set; } = "1.0";
        public ProjectInfoV1 Project { get; set; } = new ProjectInfoV1();

        public StoreConfigV1 Config { get; set; } = new StoreConfigV1();

        // System counters (keys are system names; use case-insensitive dictionary when constructing)
        //public Dictionary<string, SystemCounterV1> Systems { get; set; }
        //    = new Dictionary<string, SystemCounterV1>(StringComparer.OrdinalIgnoreCase);

        public System.Collections.Generic.Dictionary<string, SystemCounterV1> Systems { get; set; }
            = new System.Collections.Generic.Dictionary<string, SystemCounterV1>(System.StringComparer.OrdinalIgnoreCase);

        // Active tags
        //public List<TagRecordV1> Tags { get; set; } = new List<TagRecordV1>();
        public System.Collections.Generic.List<TagRecordV1> Tags { get; set; }
            = new System.Collections.Generic.List<TagRecordV1>();

        // Pool of released IDs (eligible for reuse)
        //public List<TagReleaseRecordV1> Released { get; set; } = new List<TagReleaseRecordV1>();
        public System.Collections.Generic.List<TagReleaseRecordV1> Released { get; set; }
           = new System.Collections.Generic.List<TagReleaseRecordV1>();

        // Global history for store-level actions
        //public List<HistoryEntryV1> History { get; set; } = new List<HistoryEntryV1>();
        public System.Collections.Generic.List<HistoryEntryV1> History { get; set; }
            = new System.Collections.Generic.List<HistoryEntryV1>();
    }
}

// File: ProjectStoreModels.cs
// v2.0.0 (C# 7.3)
using System.Collections.Generic;

namespace AutoCAD_Shared_Project.ProjectStore
{
    public sealed class ProjectStoreSnapshot
    {
        public Dictionary<string, ProjectEntry> Projects { get; set; } = new Dictionary<string, ProjectEntry>();
        public string ActiveProjectKey { get; set; }
    }

    public sealed class ProjectEntry
    {
        public string ProjectKey { get; set; }
        public string RootHint { get; set; } // absolute path or UNC
        public List<DrawingEntry> Drawings { get; set; } = new List<DrawingEntry>();
        public TaggingEntry Tagging { get; set; } = new TaggingEntry();
        public BlocksEntry Blocks { get; set; } = new BlocksEntry();
    }

    public sealed class DrawingEntry { public string Path { get; set; } }

    public sealed class TaggingEntry
    {
        public TagChannel Line { get; set; } = new TagChannel { Prefix = "L-", NextNumber = 1000, Padding = 0 };
        public TagChannel Valve { get; set; } = new TagChannel { Prefix = "V-", NextNumber = 1, Padding = 0 };
    }

    public sealed class TagChannel
    {
        public string Prefix { get; set; }
        public int NextNumber { get; set; }
        public int Padding { get; set; }
    }

    public sealed class BlocksEntry
    {
        public ValveBlocks Valve { get; set; } = new ValveBlocks();
    }

    public sealed class ValveBlocks
    {
        public Dictionary<string, ValveTypeRef> Types { get; set; } = DefaultValveTypes();
        public ValveSizeMap SizeMap { get; set; } = new ValveSizeMap
        {
            Units = "in",
            DynamicProperty = "SIZE",
            NominalToBlockScale = new Dictionary<string, double> { { "0.5", 0.5 }, { "1", 1.0 }, { "2", 2.0 } }
        };

        static Dictionary<string, ValveTypeRef> DefaultValveTypes()
        {
            return new Dictionary<string, ValveTypeRef>
            {
                { "Ball",      new ValveTypeRef { Name = "INST_BALL-VALVE", Path = @"C:\CBICAD\AutoCAD\Blocks\Instrumentation\INST_BALL-VALVE.dwg" } },
                { "Butterfly", new ValveTypeRef { Name = "INST_BFLY-VALVE",  Path = @"C:\CBICAD\AutoCAD\Blocks\Instrumentation\INST_BFLY-VALVE.dwg" } },
                { "Check",     new ValveTypeRef { Name = "VALVE_CHK",  Path = @"C:\Libraries\valves.dwg" } },
            };
        }
    }

    public sealed class ValveTypeRef { public string Name { get; set; } public string Path { get; set; } }

    public sealed class ValveSizeMap
    {
        public string Units { get; set; }              // "in" | "mm"
        public string DynamicProperty { get; set; }    // e.g., "SIZE"
        public Dictionary<string, double> NominalToBlockScale { get; set; } = new Dictionary<string, double>();
    }
}

using System;
using System.Collections.Generic;
using System.IO;
using System.Text;
using System.Threading;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

namespace AutoCAD_Shared_Project.ProjectStore
{
    public static partial class ProjectStoreService
    {
        public static V1.ProjectStoreV1 Load(string path)
        {
            if (string.IsNullOrWhiteSpace(path))
                throw new ArgumentException("Project store path is null or empty.", nameof(path));

            if (!File.Exists(path))
                throw new FileNotFoundException("Project store file not found.", path);

            var json = File.ReadAllText(path);
            var store = JsonConvert.DeserializeObject<V1.ProjectStoreV1>(json);

            if (store == null)
                throw new InvalidOperationException($"Failed to deserialize project store at '{path}'.");

            try
            {
                // Full validation when possible (v1 store with Systems defined etc.)
                ProjectStoreValidator.ValidateOrThrow(store);
            }
            catch (InvalidOperationException)
            {
                // IMPORTANT:
                // For now we tolerate legacy / partial files (e.g. old registry JSON
                // with no Systems). Callers like GetSystemNextList(s) already
                // null-check s.Systems and will just return an empty list.
                //
                // This prevents the palette from JIT-crashing while we’re
                // in the middle of migrating everything over to v1.
            }

            return store;
        }

        public static void SaveAtomic(string path, V1.ProjectStoreV1 store)
        {
            if (string.IsNullOrWhiteSpace(path))
                throw new ArgumentNullException(nameof(path));
            if (store == null)
                throw new ArgumentNullException(nameof(store));

            // Validate before writing
            ProjectStoreValidator.ValidateOrThrow(store);

            var tmp = path + ".tmp";

            var json = JsonConvert.SerializeObject(
                store,
                Formatting.Indented,
                new JsonSerializerSettings
                {
                    NullValueHandling = NullValueHandling.Ignore
                });

            File.WriteAllText(tmp, json, Encoding.UTF8);

            if (File.Exists(path))
                File.Replace(tmp, path, null);
            else
                File.Move(tmp, path);
        }

        public static string ReadGlobalValvePrefix(string path)
        {
            if (string.IsNullOrWhiteSpace(path) || !File.Exists(path))
                return "V-"; // safe default

            var json = File.ReadAllText(path);
            var root = JObject.Parse(json);

            // --- Case 1: legacy Project Registry format ---
            // {
            //   "Projects": { "Project 6": { "Tagging": { "Valve": { "Prefix": "V-" } } } },
            //   "ActiveProjectKey": "Project 6"
            // }
            JToken projectsToken;
            if (root.TryGetValue("Projects", out projectsToken))
            {
                var activeKey = (string)root["ActiveProjectKey"];
                if (!string.IsNullOrEmpty(activeKey))
                {
                    var proj = projectsToken[activeKey];
                    if (proj != null)
                    {
                        var prefix = (string)proj["Tagging"]?["Valve"]?["Prefix"];
                        if (!string.IsNullOrEmpty(prefix))
                            return prefix;
                    }
                }

                // We’re in registry mode but can’t find a prefix: default
                return "V-";
            }

            // --- Case 2: v1 Project_Store.json format ---
            // We now support both the older object-style Tags and the newer array style.
            // Prefer Systems / Config for the valve prefix.

            var systemsObj = root["Systems"] as JObject;
            if (systemsObj != null)
            {
                var valveSys = systemsObj["VALVE"];
                if (valveSys != null && valveSys["Prefix"] != null)
                {
                    var prefix = (string)valveSys["Prefix"];
                    if (!string.IsNullOrEmpty(prefix))
                        return prefix;
                }
            }

            // Older v1 shape where Tags was an object
            var tagsObj = root["Tags"] as JObject;
            if (tagsObj != null)
            {
                var valveNode = tagsObj["Valve"];
                if (valveNode != null && valveNode["Prefix"] != null)
                {
                    var prefix = (string)valveNode["Prefix"];
                    if (!string.IsNullOrEmpty(prefix))
                        return prefix;
                }
            }

            // Newer v1 shape: Config.GlobalValvePrefix
            var cfgObj = root["Config"] as JObject;
            if (cfgObj != null && cfgObj["GlobalValvePrefix"] != null)
            {
                var prefix = (string)cfgObj["GlobalValvePrefix"];
                if (!string.IsNullOrEmpty(prefix))
                    return prefix;
            }

            // Final fallback
            return "V-";
        }

        // Similar pattern for ReadGraceDays and ReadPreferReleasedLowestFirst:
        //  - Look in legacy if "Projects" exists
        //  - Look in v1 store (maybe under Config or Project) otherwise
        public static int ReadGraceDays(string path)
        {
            if (string.IsNullOrWhiteSpace(path) || !File.Exists(path))
                return 0;

            var json = File.ReadAllText(path);
            var root = JObject.Parse(json);

            // Legacy: maybe stored under project settings (if you had it)
            if (root["Projects"] != null)
            {
                var activeKey = (string)root["ActiveProjectKey"];
                if (!string.IsNullOrEmpty(activeKey))
                {
                    var proj = root["Projects"][activeKey];
                    if (proj != null && proj["GraceDays"] != null)
                        return (int)proj["GraceDays"];
                }

                return 0; // default
            }

            // v1: maybe under Config or Project – adjust to your schema
            var cfg = root["Config"];
            if (cfg != null && cfg["GraceDays"] != null)
                return (int)cfg["GraceDays"];

            return 0;
        }

        public static bool ReadPreferReleasedLowestFirst(string path)
        {
            if (string.IsNullOrWhiteSpace(path) || !File.Exists(path))
                return false;

            var json = SafeReadAllText(path);
            var root = JObject.Parse(json);

            if (root["Projects"] != null)
            {
                var activeKey = (string)root["ActiveProjectKey"];
                if (!string.IsNullOrEmpty(activeKey))
                {
                    var proj = root["Projects"][activeKey];
                    if (proj != null && proj["PreferReleasedLowestFirst"] != null)
                        return (bool)proj["PreferReleasedLowestFirst"];
                }

                return false;
            }

            var cfg = root["Config"];
            if (cfg != null && cfg["PreferReleasedLowestFirst"] != null)
                return (bool)cfg["PreferReleasedLowestFirst"];

            return false;
        }

        // ---------- DTO used by TagNumbersDynamic ----------
        // NOTE: TagNumbersDynamic expects Name/SystemKey/Next.
        public sealed class SystemNextInfo
        {
            // Legacy/UI fields (keep all for widest compatibility)
            public string System { get; set; }      // used by some panes
            public string Name { get; set; }        // optional display
            public string SystemKey { get; set; }   // dictionary key

            public string Prefix { get; set; }
            public long Next { get; set; }
            public int End { get; set; }
        }

        // ------ Path overloads for WRITE (load → mutate → save) -----------------

        public static void WriteGlobalValvePrefix(string path, string prefix)
        {
            if (string.IsNullOrWhiteSpace(path) || !File.Exists(path))
                return;

            try
            {
                var store = Load(path);                // V1.ProjectStoreV1
                WriteGlobalValvePrefix(store, prefix); // object-based
                SaveAtomic(path, store);
            }
            catch
            {
                // For now: ignore if path is legacy registry JSON, not a v1 store
            }
        }

        public static void WriteGraceDays(string path, int days)
        {
            if (string.IsNullOrWhiteSpace(path) || !File.Exists(path))
                return;

            try
            {
                var store = Load(path);
                WriteGraceDays(store, days);
                SaveAtomic(path, store);
            }
            catch { }
        }

        public static void WritePreferReleasedLowestFirst(string path, bool value)
        {
            if (string.IsNullOrWhiteSpace(path) || !File.Exists(path))
                return;

            try
            {
                var store = Load(path);
                WritePreferReleasedLowestFirst(store, value);
                SaveAtomic(path, store);
            }
            catch { }
        }

        private static string SafeReadAllText(string path)
        {
            if (string.IsNullOrWhiteSpace(path))
                throw new ArgumentNullException(nameof(path));

            const int maxAttempts = 5;
            const int delayMs = 100;

            for (int attempt = 0; attempt < maxAttempts; attempt++)
            {
                try
                {
                    // Use shared read so OneDrive or other processes don't block us
                    using (var fs = new FileStream(
                        path,
                        FileMode.Open,
                        FileAccess.Read,
                        FileShare.ReadWrite | FileShare.Delete))
                    using (var reader = new StreamReader(fs, Encoding.UTF8, true))
                    {
                        return reader.ReadToEnd();
                    }
                }
                catch (IOException)
                {
                    // Last attempt? rethrow
                    if (attempt == maxAttempts - 1)
                        throw;

                    // Brief pause to let the other process finish touching the file
                    Thread.Sleep(delayMs);
                }
            }

            // Should never hit here because we either return or throw above
            throw new IOException("Failed to read ProjectStore file after multiple attempts: " + path);
        }

// ---------- Factory / back-compat helpers for tag tools ----------

        public static V1.ProjectStoreV1 CreateNewStore(string projectName = "New Project")
        {
            return new V1.ProjectStoreV1
            {
                Version = "1.0",
                Project = new V1.ProjectInfoV1 { Name = projectName },
                Config = new V1.StoreConfigV1
                {
                    GlobalValvePrefix = "V",
                    GraceDays = 7,
                    PreferReleasedLowestFirst = true
                },
                Systems = new System.Collections.Generic.Dictionary<string, V1.SystemCounterV1>(StringComparer.OrdinalIgnoreCase)
        {
            { "GENERAL", new V1.SystemCounterV1 { Prefix = "INST", Pad = 4, Start = 1, End = 9999, Next = 1 } }
        },
                Tags = new System.Collections.Generic.List<V1.TagRecordV1>(),
                Released = new System.Collections.Generic.List<V1.TagReleaseRecordV1>(),
                History = new System.Collections.Generic.List<V1.HistoryEntryV1>()
            };
        }

        // Back-compat: older code calls Save(...) directly.
        public static void Save(string path, V1.ProjectStoreV1 store)
        {
            // Delegate to the new atomic writer
            SaveAtomic(path, store);
        }

        // Back-compat: older code calls CreateDefault(...) when seeding a file.
        public static V1.ProjectStoreV1 CreateDefault(string projectName = "New Project")
        {
            return CreateNewStore(projectName);
        }

        //public static IList<SystemNextSnapshot> GetSystemNextList(string path)
        //{
        //    var list = new List<SystemNextSnapshot>();

        //    if (string.IsNullOrWhiteSpace(path) || !File.Exists(path))
        //        return list;

        //    var json = File.ReadAllText(path);
        //    var root = JObject.Parse(json);

        //    // Legacy: no real "Systems" object, so fake it from Tagging
        //    if (root["Projects"] != null)
        //    {
        //        var activeKey = (string)root["ActiveProjectKey"];
        //        if (!string.IsNullOrEmpty(activeKey))
        //        {
        //            var proj = root["Projects"][activeKey];
        //            if (proj != null)
        //            {
        //                var line = proj["Tagging"]?["Line"];
        //                if (line != null)
        //                {
        //                    list.Add(new SystemNextSnapshot
        //                    {
        //                        Key = "LINE",
        //                        Name = "Line Numbering",
        //                        Prefix = (string)line["Prefix"] ?? "L-",
        //                        NextNumber = (int?)line["NextNumber"] ?? 1,
        //                        Pad = (int?)line["Padding"] ?? 0
        //                    });
        //                }

        //                var valve = proj["Tagging"]?["Valve"];
        //                if (valve != null)
        //                {
        //                    list.Add(new SystemNextSnapshot
        //                    {
        //                        Key = "VALVE",
        //                        Name = "Valve Numbering",
        //                        Prefix = (string)valve["Prefix"] ?? "V-",
        //                        NextNumber = (int?)valve["NextNumber"] ?? 1,
        //                        Pad = (int?)valve["Padding"] ?? 0
        //                    });
        //                }
        //            }
        //        }

        //        return list;
        //    }

        //    // v1: real Systems dictionary
        //    var systemsToken = root["Systems"] as JObject;
        //    if (systemsToken != null)
        //    {
        //        foreach (var kvp in systemsToken)
        //        {
        //            var sysObj = kvp.Value as JObject;
        //            if (sysObj == null) continue;

        //            var snap = new SystemNextSnapshot
        //            {
        //                Key = (string)sysObj["Key"] ?? kvp.Key,
        //                Name = (string)sysObj["Name"] ?? kvp.Key,
        //                Prefix = (string)sysObj["Prefix"] ?? string.Empty,
        //                NextNumber = (int?)sysObj["Next"] ?? 0,
        //                Pad = (int?)sysObj["Pad"] ?? 0
        //            };
        //            list.Add(snap);
        //        }
        //    }

        //    return list;
        //}
    }
}

// ----------------------------------------------------------------------------
// Minimal validator + repair helpers (v1)
// ----------------------------------------------------------------------------
using System;
using System.Linq;
using V1 = AutoCAD_Shared_Project.ProjectStore.V1;

namespace AutoCAD_Shared_Project.ProjectStore
{
    public static class ProjectStoreValidator
    {
        public static void ValidateOrThrow(V1.ProjectStoreV1 s)
        {
            if (s == null) throw new ArgumentNullException(nameof(s));
            if (string.IsNullOrWhiteSpace(s.Version)) throw new InvalidOperationException("Version missing.");
            if (s.Config == null) s.Config = new V1.StoreConfigV1();
            if (s.Systems == null || s.Systems.Count == 0)
                throw new InvalidOperationException("No Systems defined.");
            foreach (var kv in s.Systems)
            {
                var c = kv.Value ?? new V1.SystemCounterV1();
                if (c.End < c.Start) throw new InvalidOperationException($"System '{kv.Key}' has End < Start.");
            }
            // ensure lists
            s.Tags = s.Tags ?? new System.Collections.Generic.List<V1.TagRecordV1>();
            s.Released = s.Released ?? new System.Collections.Generic.List<V1.TagReleaseRecordV1>();
            s.History = s.History ?? new System.Collections.Generic.List<V1.HistoryEntryV1>();
        }

        public static bool Validate(V1.ProjectStoreV1 store)
        {
            ValidateOrThrow(store);
            return true;
        }

        // Some callers referenced "RepairCommon" earlier; provide a safe no-op fixer.
        public static void RepairCommon(V1.ProjectStoreV1 s)
        {
            if (s == null) return;
            if (s.Config == null) s.Config = new V1.StoreConfigV1();
            s.Tags = s.Tags ?? new System.Collections.Generic.List<V1.TagRecordV1>();
            s.Released = s.Released ?? new System.Collections.Generic.List<V1.TagReleaseRecordV1>();
            s.History = s.History ?? new System.Collections.Generic.List<V1.HistoryEntryV1>();
        }
    }
}

// File: StoreGuards.cs
// Project: AutoCAD_Shared_Project.ProjectStore
// Version: 2.0.0 (NOD key fix + non-interactive resolve)
// Language: C# 7.3+

using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;
using V1 = AutoCAD_Shared_Project.ProjectStore.V1;

namespace AutoCAD_Shared_Project.ProjectStore
{
    public static class StoreGuards
    {
        // NEW canonical, AutoCAD-legal key for binding the ProjectStore path
        private const string NodKeyCanonical = "CBI.ProjectStore.Path";

        // LEGACY key (invalid in practice, but kept for defensive migration)
        private const string NodKeyLegacy = "CBI/ProjectStore/Path";

        #region High-level path resolution (interactive)

        /// <summary>
        /// Interactive path resolver for tools that are allowed to show UI.
        /// 1) Tries to resolve the ProjectStore path from the NOD (canonical + migration).
        /// 2) If not bound, prompts the user to pick ProjectStore.json.
        /// 3) Binds the drawing to that path via the NOD.
        /// </summary>
        public static bool TryResolvePath(Database db, Editor ed, out string path)
        {
            path = null;
            if (db == null)
            {
                ed?.WriteMessage("\n[StoreGuards] Database is null.");
                return false;
            }

            // First, try non-interactive resolution (canonical + migration)
            string error;
            if (TryResolvePath(db, out path, out error, validateFileExists: true))
            {
                return true;
            }

            // If a specific error occurred (e.g., file missing), show it but still allow user to pick a new file.
            if (!string.IsNullOrWhiteSpace(error))
            {
                ed?.WriteMessage("\n[StoreGuards] " + error);
            }

            // Fallback: prompt user to select a ProjectStore JSON file
            var p = new PromptOpenFileOptions("\nSelect ProjectStore.json")
            {
                Filter = "JSON (*.json)|*.json|All Files (*.*)|*.*"
            };
            var r = ed.GetFileNameForOpen(p);
            if (r.Status != PromptStatus.OK)
                return false;

            path = r.StringResult;

            // Bind this drawing to the selected path
            string bindError;
            if (!TryWritePath(db, path, out bindError))
            {
                ed?.WriteMessage("\n[StoreGuards] Failed to bind drawing to ProjectStore: " + bindError);
                path = null;
                return false;
            }

            return true;
        }

        #endregion

        #region Non-interactive path resolution (for services/tools)

        /// <summary>
        /// Non-interactive resolver.
        /// - Looks up the ProjectStore path in the NOD under the canonical key.
        /// - If not found, optionally migrates a legacy key.
        /// - Optionally validates that the file exists.
        /// Never throws; returns false on any error.
        /// </summary>
        public static bool TryResolvePath(
            Database db,
            out string storePath,
            out string errorMessage,
            bool validateFileExists = true)
        {
            storePath = null;
            errorMessage = null;

            if (db == null)
            {
                errorMessage = "Database is null.";
                return false;
            }

            try
            {
                using (var tr = db.TransactionManager.StartTransaction())
                {
                    var nod = (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId, OpenMode.ForRead);

                    // 1) Try canonical key first
                    if (TryReadPathFromKey(nod, tr, NodKeyCanonical, out storePath))
                    {
                        if (validateFileExists && !File.Exists(storePath))
                        {
                            errorMessage = "ProjectStore path is stored, but the file does not exist: " + storePath;
                            storePath = null;
                            return false;
                        }

                        return true;
                    }

                    // 2) Try legacy key and migrate (defensive / future-proof)
                    string legacyPath;
                    if (nod.Contains(NodKeyLegacy) &&
                        TryReadPathFromKey(nod, tr, NodKeyLegacy, out legacyPath) &&
                        !string.IsNullOrWhiteSpace(legacyPath))
                    {
                        nod.UpgradeOpen();

                        Xrecord xrec;
                        if (nod.Contains(NodKeyCanonical))
                        {
                            xrec = (Xrecord)tr.GetObject(nod.GetAt(NodKeyCanonical), OpenMode.ForWrite);
                        }
                        else
                        {
                            xrec = new Xrecord();
                            nod.SetAt(NodKeyCanonical, xrec);
                            tr.AddNewlyCreatedDBObject(xrec, true);
                        }

                        xrec.Data = new ResultBuffer(
                            new TypedValue((int)DxfCode.Text, legacyPath ?? string.Empty)
                        );

                        // Optional: remove legacy key
                        if (nod.Contains(NodKeyLegacy))
                        {
                            nod.Remove(NodKeyLegacy);
                        }

                        tr.Commit();

                        if (validateFileExists && !File.Exists(legacyPath))
                        {
                            errorMessage = "Migrated legacy ProjectStore path, but file does not exist: " + legacyPath;
                            return false;
                        }

                        storePath = legacyPath;
                        return true;
                    }

                    // 3) No binding at all
                    errorMessage = "Drawing is not bound to a ProjectStore.";
                    return false;
                }
            }
            catch (Autodesk.AutoCAD.Runtime.Exception ex)
            {
                errorMessage = "Failed to resolve ProjectStore path (AutoCAD error): " + ex.Message;
                storePath = null;
                return false;
            }
            catch (System.Exception ex)
            {
                errorMessage = "Unexpected error while resolving ProjectStore path: " + ex.Message;
                storePath = null;
                return false;
            }
        }

        /// <summary>
        /// Convenience overload: non-interactive resolve without error details.
        /// </summary>
        public static bool TryResolvePath(Database db, out string storePath)
        {
            string error;
            return TryResolvePath(db, out storePath, out error, validateFileExists: true);
        }

        #endregion

        #region ProjectStore open / prompt / save

        public static V1.ProjectStoreV1 OpenReadOnly(string path)
            => ProjectStoreService.Load(path);

        public static string PromptForSystemKey(Editor ed, V1.ProjectStoreV1 store)
        {
            var keys = (store?.Systems != null) ? new List<string>(store.Systems.Keys) : new List<string>();
            if (keys.Count == 0) return null;
            if (keys.Count == 1) return keys[0];

            var p = new PromptKeywordOptions("\nSelect System") { AllowNone = false };
            foreach (var k in keys) p.Keywords.Add(k);
            var r = ed.GetKeywords(p);
            return r.Status == PromptStatus.OK ? r.StringResult : null;
        }

        public static bool SafeSave(string path, V1.ProjectStoreV1 store, Editor ed = null)
        {
            try
            {
                ProjectStoreService.Save(path, store);
                return true;
            }
            catch (System.Exception ex)
            {
                ed?.WriteMessage($"\nFailed to save ProjectStore: {ex.Message}");
                return false;
            }
        }

        #endregion

        #region Internal NOD helpers

        /// <summary>
        /// Older helper used by interactive resolver. Now delegates to non-interactive logic
        /// without file-existence validation.
        /// </summary>
        private static string TryReadPath(Database db)
        {
            string path;
            string error;
            if (TryResolvePath(db, out path, out error, validateFileExists: false))
                return path;

            return null;
        }

        /// <summary>
        /// New safe binder that uses the canonical key and shields callers from AutoCAD exceptions.
        /// </summary>
        public static bool TryWritePath(Database db, string path, out string errorMessage)
        {
            errorMessage = null;

            if (db == null)
            {
                errorMessage = "Database is null.";
                return false;
            }

            if (string.IsNullOrWhiteSpace(path))
            {
                errorMessage = "ProjectStore path is empty.";
                return false;
            }

            try
            {
                using (var tr = db.TransactionManager.StartTransaction())
                {
                    var nod = (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId, OpenMode.ForWrite);

                    Xrecord xr;
                    if (nod.Contains(NodKeyCanonical))
                    {
                        xr = (Xrecord)tr.GetObject(nod.GetAt(NodKeyCanonical), OpenMode.ForWrite);
                    }
                    else
                    {
                        xr = new Xrecord();
                        nod.SetAt(NodKeyCanonical, xr); // legal key
                        tr.AddNewlyCreatedDBObject(xr, true);
                    }

                    xr.Data = new ResultBuffer(new TypedValue((int)DxfCode.Text, path ?? string.Empty));
                    tr.Commit();
                }

                return true;
            }
            catch (Autodesk.AutoCAD.Runtime.Exception ex)
            {
                errorMessage = "Failed to bind drawing to ProjectStore (AutoCAD error): " + ex.Message;
                return false;
            }
            catch (System.Exception ex)
            {
                errorMessage = "Unexpected error while binding ProjectStore: " + ex.Message;
                return false;
            }
        }

        /// <summary>
        /// Backward-compatible wrapper for older callers that ignored errors.
        /// </summary>
        private static void TryWritePath(Database db, string path)
        {
            string err;
            TryWritePath(db, path, out err);
            // Errors intentionally ignored here (legacy behavior).
        }

        /// <summary>
        /// Low-level Xrecord reader used by non-interactive resolution.
        /// </summary>
        private static bool TryReadPathFromKey(
            DBDictionary nod,
            Transaction tr,
            string key,
            out string path)
        {
            path = null;

            if (nod == null || tr == null)
                return false;

            if (!nod.Contains(key))
                return false;

            var id = nod.GetAt(key);
            var xrec = tr.GetObject(id, OpenMode.ForRead) as Xrecord;
            if (xrec == null || xrec.Data == null)
                return false;

            foreach (TypedValue tv in xrec.Data)
            {
                if (tv.TypeCode == (int)DxfCode.Text ||
                    tv.TypeCode == (int)DxfCode.ExtendedDataAsciiString)
                {
                    var value = tv.Value as string;
                    if (!string.IsNullOrWhiteSpace(value))
                    {
                        path = value;
                        return true;
                    }
                }
            }

            return false;
        }

        #endregion

        #region Tag guard helpers (unchanged)

        public static int AutoReleaseAllEligible(V1.ProjectStoreV1 store, string reason, string by)
        {
            if (store == null) return 0;
            var orphans = store.Tags.Where(t => t.Status == V1.TagStatusV1.Orphan).ToList();
            var count = 0;
            foreach (var t in orphans)
            {
                try
                {
                    if (TagStoreV1.Release(store, t.Id, reason ?? "AutoRelease", by ?? Environment.UserName))
                        count++;
                }
                catch
                {
                    // ignore invalid states
                }
            }
            return count;
        }

        // path overloads
        public static bool TagIdExists(string storePath, string id)
        {
            if (string.IsNullOrWhiteSpace(storePath) || string.IsNullOrWhiteSpace(id)) return false;
            var s = ProjectStoreService.Load(storePath);
            return TagIdExists(s, id);
        }

        public static bool TagIdOwnedByParent(string storePath, string id, string parentHandle)
        {
            if (string.IsNullOrWhiteSpace(storePath)) return false;
            var s = ProjectStoreService.Load(storePath);
            return TagIdOwnedByParent(s, id, parentHandle);
        }

        public static bool ChildHandleExists(string storePath, string childHandle)
        {
            // v1.0 TagStoreV1 schema does NOT track child bubbles centrally.
            // For now, always return false (prevents false “duplicate bubble” blocks).
            return false;
        }

        // store overloads
        public static bool TagIdExists(V1.ProjectStoreV1 store, string id)
        {
            if (store?.Tags == null || string.IsNullOrWhiteSpace(id)) return false;
            return store.Tags.Any(t => string.Equals(t.Id, id, StringComparison.OrdinalIgnoreCase));
        }

        public static bool TagIdOwnedByParent(V1.ProjectStoreV1 store, string id, string parentHandle)
        {
            if (store?.Tags == null || string.IsNullOrWhiteSpace(id)) return false;
            return store.Tags.Any(t =>
                string.Equals(t.Id, id, StringComparison.OrdinalIgnoreCase) &&
                string.Equals(t.EntityHandle ?? "", parentHandle ?? "", StringComparison.OrdinalIgnoreCase));
        }

        #endregion
    }
}

// File: ProjectSettingsService.cs
// v2.0.3 (C# 7.3)
using System;
using System.Collections.Generic;
using System.IO;
using Newtonsoft.Json;

namespace AutoCAD_Shared_Project.ProjectStore
{
    public sealed class ProjectSettingsService : IProjectSettingsService_V2
    {
        private readonly string _storePath;
        public ProjectSettingsService(string storePath) { _storePath = storePath; }

        public ProjectStoreSnapshot Read()
        {
            if (!File.Exists(_storePath)) return new ProjectStoreSnapshot();
            var json = File.ReadAllText(_storePath);
            var snap = JsonConvert.DeserializeObject<ProjectStoreSnapshot>(json) ?? new ProjectStoreSnapshot();
            // No validator calls here to avoid V1 type mismatches.
            return snap;
        }

        public void Write(ProjectStoreSnapshot snapshot)
        {
            var json = JsonConvert.SerializeObject(snapshot, Formatting.Indented);
            Directory.CreateDirectory(Path.GetDirectoryName(_storePath) ?? ".");
            File.WriteAllText(_storePath, json);
        }

        public void CreateNewProject(string projectKey, string rootHint, bool scanForDrawings)
        {
            if (string.IsNullOrWhiteSpace(projectKey))
                throw new ArgumentException("projectKey required", nameof(projectKey));
            if (string.IsNullOrWhiteSpace(rootHint))
                throw new ArgumentException("rootHint required", nameof(rootHint));

            var snap = Read();
            if (snap.Projects == null) snap.Projects = new Dictionary<string, ProjectEntry>();
            if (snap.Projects.ContainsKey(projectKey))
                throw new InvalidOperationException("Project already exists: " + projectKey);

            var normalizedRoot = NewProjectBuilder.NormalizePath(rootHint);
            IEnumerable<string> drawings = scanForDrawings
                ? NewProjectBuilder.DiscoverDwgs(normalizedRoot)
                : new string[0];

            var entry = NewProjectBuilder.Create(projectKey, normalizedRoot, drawings);
            if (entry.Tagging == null) entry.Tagging = new TaggingEntry();
            if (entry.Blocks == null) entry.Blocks = new BlocksEntry();

            snap.Projects[projectKey] = entry;
            snap.ActiveProjectKey = projectKey;

            Write(snap);
        }

        public string GetValvePrefix()
        {
            var s = Read();
            ProjectEntry p;
            return (s.Projects != null && s.Projects.TryGetValue(GetActive(), out p)) ? (p.Tagging.Valve.Prefix ?? "V-") : "V-";
        }

        public void SetValvePrefix(string prefix)
        {
            var s = Read();
            var key = GetActiveOrThrow(s);
            s.Projects[key].Tagging.Valve.Prefix = string.IsNullOrWhiteSpace(prefix) ? "V-" : prefix;
            Write(s);
        }

        public string GetLinePrefix()
        {
            var s = Read();
            ProjectEntry p;
            return (s.Projects != null && s.Projects.TryGetValue(GetActive(), out p)) ? (p.Tagging.Line.Prefix ?? "L-") : "L-";
        }

        public void SetLinePrefix(string prefix)
        {
            var s = Read();
            var key = GetActiveOrThrow(s);
            s.Projects[key].Tagging.Line.Prefix = string.IsNullOrWhiteSpace(prefix) ? "L-" : prefix;
            Write(s);
        }

        public int PeekNextValveNumber()
        {
            var s = Read(); var k = GetActiveOrThrow(s); return s.Projects[k].Tagging.Valve.NextNumber;
        }
        public int PeekNextLineNumber()
        {
            var s = Read(); var k = GetActiveOrThrow(s); return s.Projects[k].Tagging.Line.NextNumber;
        }

        public string AllocateValveTag()
        {
            var s = Read(); var k = GetActiveOrThrow(s);
            var ch = s.Projects[k].Tagging.Valve;
            var pad = ch.Padding > 0 ? ch.Padding : 0;
            var tag = ch.Prefix + ch.NextNumber.ToString(pad > 0 ? new string('0', pad) : "0");
            ch.NextNumber++;
            Write(s);
            return tag;
        }

        public string AllocateLineTag()
        {
            var s = Read(); var k = GetActiveOrThrow(s);
            var ch = s.Projects[k].Tagging.Line;
            var pad = ch.Padding > 0 ? ch.Padding : 0;
            var tag = ch.Prefix + ch.NextNumber.ToString(pad > 0 ? new string('0', pad) : "0");
            ch.NextNumber++;
            Write(s);
            return tag;
        }

        private string GetActive() => Read().ActiveProjectKey;

        private static string GetActiveOrThrow(ProjectStoreSnapshot s)
        {
            if (string.IsNullOrWhiteSpace(s.ActiveProjectKey)) throw new InvalidOperationException("No active project.");
            return s.ActiveProjectKey;
        }

        public void EnsureProject(string projectKey, string rootHint)
        {
            if (string.IsNullOrWhiteSpace(projectKey)) throw new ArgumentException("projectKey required");
            if (string.IsNullOrWhiteSpace(rootHint)) throw new ArgumentException("rootHint required");

            var snap = Read();
            if (snap.Projects == null) snap.Projects = new Dictionary<string, ProjectEntry>();

            if (!snap.Projects.ContainsKey(projectKey))
            {
                var entry = NewProjectBuilder.Create(projectKey, NewProjectBuilder.NormalizePath(rootHint));
                if (entry.Tagging == null) entry.Tagging = new TaggingEntry();
                if (entry.Blocks == null) entry.Blocks = new BlocksEntry();

                snap.Projects[projectKey] = entry;
                snap.ActiveProjectKey = projectKey;
                Write(snap);
            }
            else if (string.IsNullOrWhiteSpace(snap.ActiveProjectKey))
            {
                snap.ActiveProjectKey = projectKey;
                Write(snap);
            }
        }

        public bool AddDrawingIfMissing(string projectKey, string drawingPath)
        {
            if (string.IsNullOrWhiteSpace(projectKey)) throw new ArgumentException("projectKey required");
            if (string.IsNullOrWhiteSpace(drawingPath)) return false;

            var snap = Read();
            if (snap.Projects == null || !snap.Projects.ContainsKey(projectKey)) return false;

            var norm = NewProjectBuilder.NormalizePath(drawingPath);
            var list = snap.Projects[projectKey].Drawings ?? (snap.Projects[projectKey].Drawings = new List<DrawingEntry>());

            foreach (var d in list)
                if (string.Equals(d?.Path, norm, StringComparison.OrdinalIgnoreCase))
                    return false;

            list.Add(new DrawingEntry { Path = norm });
            Write(snap);
            return true;
        }

        public bool EnsureProjectAndAddDrawing(string projectKey, string rootHint, string drawingPath)
        {
            EnsureProject(projectKey, rootHint);
            return AddDrawingIfMissing(projectKey, drawingPath);
        }
    }

    public interface IProjectSettingsService_V2
    {
        ProjectStoreSnapshot Read();
        void Write(ProjectStoreSnapshot snapshot);
        void CreateNewProject(string projectKey, string rootHint, bool scanForDrawings);

        string GetValvePrefix();
        void SetValvePrefix(string prefix);
        string GetLinePrefix();
        void SetLinePrefix(string prefix);
        int PeekNextValveNumber();
        int PeekNextLineNumber();

        string AllocateValveTag();
        string AllocateLineTag();
    }
}

// v2.0.0 (C# 7.3)
using System;
using System.Collections.Generic;
using System.IO;

namespace AutoCAD_Shared_Project.ProjectStore
{
    public static class NewProjectBuilder
    {
        public static ProjectEntry Create(string projectKey, string rootHint, IEnumerable<string> seedDrawingPaths = null)
        {
            if (string.IsNullOrWhiteSpace(projectKey)) throw new ArgumentException("projectKey required");
            if (string.IsNullOrWhiteSpace(rootHint)) throw new ArgumentException("rootHint required");

            var normalizedRoot = NormalizePath(rootHint);

            var entry = new ProjectEntry
            {
                ProjectKey = projectKey.Trim(),
                RootHint = normalizedRoot,
                Drawings = new List<DrawingEntry>()
            };

            if (seedDrawingPaths != null)
            {
                foreach (var p in seedDrawingPaths)
                {
                    var n = NormalizePath(p);
                    if (!string.IsNullOrWhiteSpace(n))
                        entry.Drawings.Add(new DrawingEntry { Path = n });
                }
            }

            // Ensure list is never null (already set) and contains no duplicates
            Dedup(entry.Drawings);
            return entry;
        }

        public static IEnumerable<string> DiscoverDwgs(string rootHint, int max = 1000)
        {
            var results = new List<string>();
            var root = NormalizePath(rootHint);
            if (string.IsNullOrWhiteSpace(root) || !Directory.Exists(root)) return results;

            try
            {
                foreach (var f in Directory.EnumerateFiles(root, "*.dwg", SearchOption.AllDirectories))
                {
                    results.Add(NormalizePath(f));
                    if (results.Count >= max) break;
                }
            }
            catch { /* ignore filesystem races */ }

            return results;
        }

        public static string NormalizePath(string path)
        {
            if (string.IsNullOrWhiteSpace(path)) return null;
            try { return Path.GetFullPath(path); }
            catch { return path.Trim(); }
        }

        private static void Dedup(List<DrawingEntry> list)
        {
            if (list == null) return;
            var seen = new HashSet<string>(StringComparer.OrdinalIgnoreCase);
            for (int i = list.Count - 1; i >= 0; i--)
            {
                var p = list[i]?.Path ?? string.Empty;
                if (seen.Contains(p)) list.RemoveAt(i);
                else seen.Add(p);
            }
        }
    }
}

// File: AutoCAD_Shared_Project/ProjectStore/SafeStoreAllocator.cs
// C# 7.3
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;
using System.Threading;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

namespace AutoCAD_Shared_Project.ProjectStore
{
    public static class SafeStoreAllocator
    {
        public sealed class AllocResult
        {
            public string SystemKey;
            public string Prefix;
            public int Pad;
            public int Number;
            public string PrintedId; // e.g., V-1023
            public string Action;    // ALLOCATE or REUSE
        }

        // Main entry: safe allocation (mutex + file lock + atomic replace + retries)
        public static AllocResult AllocateNextSafe(string storePath, string systemKey, string who, string drawingPath, int maxRetries = 6)
        {
            if (string.IsNullOrWhiteSpace(storePath)) throw new ArgumentNullException(nameof(storePath));
            if (string.IsNullOrWhiteSpace(systemKey)) throw new ArgumentNullException(nameof(systemKey));

            var mutexName = @"Global\CBI_Store_" + HashForName(storePath) + "_" + Sanitize(systemKey);
            using (var mutex = new Mutex(false, mutexName))
            {
                var rand = new Random();
                for (int attempt = 0; attempt < maxRetries; attempt++)
                {
                    bool lockTaken = false;
                    try
                    {
                        // Intra-process/machine guard (wait up to ~2s then retry with backoff)
                        lockTaken = mutex.WaitOne(TimeSpan.FromMilliseconds(2000));
                        if (!lockTaken) throw new IOException("Mutex contention.");

                        // Cross-process/machine guard: open with exclusive share
                        using (var fs = OpenExclusive(storePath))
                        using (var sr = new StreamReader(fs, Encoding.UTF8, true, 4096, leaveOpen: true))
                        using (var sw = new StreamWriter(fs, Encoding.UTF8, 4096, leaveOpen: true))
                        {
                            // read JSON
                            fs.Seek(0, SeekOrigin.Begin);
                            var json = sr.ReadToEnd();
                            var root = string.IsNullOrWhiteSpace(json) ? new JObject() : JObject.Parse(json);

                            // version stamp (optional)
                            var versionTick = (int?)root["VersionTick"] ?? 0;

                            // -------- allocate (REUSE or ALLOCATE) ----------
                            var res = AllocateInMemory(root, systemKey, who, drawingPath);

                            // bump version
                            root["VersionTick"] = versionTick + 1;

                            // -------- atomic save ----------
                            var tmp = storePath + ".tmp";
                            File.WriteAllText(tmp, root.ToString(Formatting.Indented), Encoding.UTF8);
                            // Replace preserves ACLs; if not available use Move after Delete
                            TryFileReplace(tmp, storePath, storePath + ".bak");

                            return res;
                        }
                    }
                    catch (IOException)
                    {
                        // brief backoff (jitter) then retry
                        Thread.Sleep(120 + rand.Next(0, 200) * (attempt + 1));
                    }
                    catch (JsonReaderException)
                    {
                        // corrupt/incomplete read due to race; back off and retry
                        Thread.Sleep(150);
                    }
                    finally
                    {
                        if (lockTaken) mutex.ReleaseMutex();
                    }
                }

                throw new IOException("Could not acquire store for allocation after multiple retries.");
            }
        }

        // In-memory allocator (v0.4 semantics)
        private static AllocResult AllocateInMemory(JObject root, string systemKey, string who, string drawingPath)
        {
            if (root["Systems"] == null) root["Systems"] = new JObject();
            if (root["Released"] == null) root["Released"] = new JObject();
            if (root["History"] == null) root["History"] = new JArray();

            var systems = (JObject)root["Systems"];
            if (!systems.ContainsKey(systemKey))
                throw new InvalidOperationException($"System '{systemKey}' not found.");

            var cfg = new StoreConfigLite(root["Config"] as JObject);

            //var sys = (JObject)systems[systemKey];
            //var prefix = (string)(sys["Prefix"] ?? "V");
            //var pad = (int?)sys["Pad"] ?? 4;

            var tagBlock = root["Tagging"] as JObject;
            var globalPrefix = (string)(tagBlock?["GlobalValvePrefix"]) ?? "V";
            var globalPad = (int?)(tagBlock?["Pad"]) ?? 4;

            var sys = (JObject)systems[systemKey];
            // per-system override still possible; otherwise inherit global
            var prefix = (string)(sys["Prefix"]) ?? globalPrefix;
            var pad = (int?)sys["Pad"] ?? globalPad;

            var start = (int?)sys["Start"] ?? 1;
            var end = (int?)sys["End"] ?? 9999;
            var next = (int?)sys["Next"] ?? start;

            var released = (JObject)root["Released"];
            var relArr = (JArray)(released[systemKey] ?? (released[systemKey] = new JArray()));

            int number;
            var action = "ALLOCATE";

            if (cfg.PreferReleasedLowestFirst && relArr.Count > 0)
            {
                // pick lowest
                int min = int.MaxValue, idx = -1;
                for (int i = 0; i < relArr.Count; i++)
                {
                    var n = (int)relArr[i];
                    if (n < min) { min = n; idx = i; }
                }
                if (idx >= 0) relArr.RemoveAt(idx);
                number = min;
                action = "REUSE";
            }
            else
            {
                if (next < start) next = start;
                if (next > end) throw new InvalidOperationException($"System '{systemKey}' exhausted ({end}).");
                number = next;
                sys["Next"] = next + 1;
            }

            var printed = $"{prefix}-{number.ToString(new string('0', pad))}";

            // history append
            var hist = (JArray)root["History"];
            hist.Add(new JObject
            {
                ["WhenUtc"] = DateTime.UtcNow.ToString("o"),
                ["Who"] = string.IsNullOrWhiteSpace(who) ? Environment.UserName : who,
                ["Action"] = action,
                ["System"] = systemKey,
                ["Number"] = number,
                ["Id"] = printed,
                ["DrawingPath"] = drawingPath ?? string.Empty
            });

            return new AllocResult
            {
                SystemKey = systemKey,
                Prefix = prefix,
                Pad = pad,
                Number = number,
                PrintedId = printed,
                Action = action
            };
        }

        // ------------- low-level helpers -------------

        private static FileStream OpenExclusive(string path)
        {
            // Ensures cross-process/cross-machine exclusion on SMB
            return new FileStream(
                path, FileMode.Open, FileAccess.ReadWrite, FileShare.None,
                bufferSize: 4096, FileOptions.None);
        }

        private static void TryFileReplace(string temp, string target, string backup)
        {
            try
            {
                // Windows atomic replace
                File.Replace(temp, target, backup, ignoreMetadataErrors: true);
            }
            catch (PlatformNotSupportedException)
            {
                // Fallback if Replace unsupported (rare)
                if (File.Exists(target)) File.Delete(target);
                File.Move(temp, target);
            }
        }

        private static string HashForName(string input)
        {
            using (var sha = SHA256.Create())
            {
                var bytes = sha.ComputeHash(Encoding.UTF8.GetBytes(input ?? ""));
                // short hash is enough for mutex name
                return BitConverter.ToString(bytes, 0, 8).Replace("-", "");
            }
        }

        private static string Sanitize(string name)
        {
            if (string.IsNullOrEmpty(name)) return "SYS";
            var sb = new StringBuilder(name.Length);
            foreach (var ch in name)
                sb.Append(char.IsLetterOrDigit(ch) ? ch : '_');
            return sb.ToString();
        }

        private sealed class StoreConfigLite
        {
            public int GraceDays { get; }
            public bool PreferReleasedLowestFirst { get; }

            public StoreConfigLite(JObject cfg)
            {
                GraceDays = (int?)cfg?["GraceDays"] ?? 7;
                PreferReleasedLowestFirst = (bool?)cfg?["PreferReleasedLowestFirst"] ?? true;
            }
        }
    }
}

using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;

namespace AutoCAD2020.ProjectStore
{
    public static class ProjectNodBinder
    {
        // Writes: NOD["CBI"]["Project"] = Xrecord( Id, Name, StorePath )
        public static void BindProjectToDrawing(Document doc, string projectId, string name, string storePath)
        {
            if (doc == null) return;
            var db = doc.Database;

            using (doc.LockDocument())
            using (var tr = db.TransactionManager.StartTransaction())
            {
                var nod = (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId, OpenMode.ForRead);

                DBDictionary cbi;
                if (nod.Contains("CBI"))
                {
                    cbi = (DBDictionary)tr.GetObject(nod.GetAt("CBI"), OpenMode.ForWrite);
                }
                else
                {
                    nod.UpgradeOpen();
                    cbi = new DBDictionary();
                    nod.SetAt("CBI", cbi);
                    tr.AddNewlyCreatedDBObject(cbi, true);
                }

                var data = new ResultBuffer(
                    new TypedValue((int)DxfCode.Text, projectId ?? string.Empty),
                    new TypedValue((int)DxfCode.Text, name ?? string.Empty),
                    new TypedValue((int)DxfCode.Text, storePath ?? string.Empty)
                );

                if (cbi.Contains("Project"))
                {
                    var xr = (Xrecord)tr.GetObject(cbi.GetAt("Project"), OpenMode.ForWrite);
                    xr.Data = data;
                }
                else
                {
                    var xr = new Xrecord { Data = data };
                    cbi.SetAt("Project", xr);
                    tr.AddNewlyCreatedDBObject(xr, true);
                }

                tr.Commit();
            }
        }
    }
}

// -----------------------------------------------------------------------------
// File: AutoCAD2020/ProjectStore/ProjectBindingService.cs
// Purpose: Resolve current project binding and raise ProjectChanged when the
//          active document's bound project/store changes.
// Target : C# 7.3 / AutoCAD 2020
// -----------------------------------------------------------------------------

using System;
using System.Collections.Generic;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;

namespace AutoCAD2020.ProjectStore
{
    public struct ProjectBinding
    {
        public string ProjectId;
        public string Name;
        public string StorePath;
        public bool IsValid => !string.IsNullOrWhiteSpace(StorePath);
    }

    public sealed class ProjectBindingChangedEventArgs : EventArgs
    {
        public Document Document { get; }
        public ProjectBinding Binding { get; }
        public string OldStorePath { get; }
        public string NewStorePath { get; }

        public ProjectBindingChangedEventArgs(Document doc, ProjectBinding binding, string oldPath, string newPath)
        {
            Document = doc;
            Binding = binding;
            OldStorePath = oldPath;
            NewStorePath = newPath;
        }
    }

    public static class ProjectBindingService
    {
        /// <summary>Raised when the binding (store path) for a document changes.</summary>
        public static event EventHandler<ProjectBindingChangedEventArgs> ProjectChanged;

        // Track last-known store path per document so we can detect changes.
        private static readonly Dictionary<Document, string> _lastStoreByDoc = new Dictionary<Document, string>();

        // Ensure we hook DM events once.
        private static bool _watching;

        /// <summary>
        /// Try to get the bound project from NOD["CBI"]["Project"] XRecord(Id, Name, StorePath).
        /// </summary>
        public static bool TryGetBoundProject(Document doc, out ProjectBinding binding)
        {
            binding = default(ProjectBinding);
            if (doc == null) return false;

            var db = doc.Database;
            try
            {
                using (var tr = db.TransactionManager.StartOpenCloseTransaction())
                {
                    var nod = (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId, OpenMode.ForRead, false, true);
                    if (nod == null || !nod.Contains("CBI")) return false;

                    var cbi = (DBDictionary)tr.GetObject(nod.GetAt("CBI"), OpenMode.ForRead);
                    if (cbi == null) return false;

                    // Primary key "Project" (modern)
                    if (cbi.Contains("Project"))
                    {
                        var xr = (Xrecord)tr.GetObject(cbi.GetAt("Project"), OpenMode.ForRead);
                        var data = xr.Data?.AsArray();
                        if (data != null && data.Length >= 3)
                        {
                            binding.ProjectId = data[0].Value as string ?? string.Empty;
                            binding.Name = data[1].Value as string ?? string.Empty;
                            binding.StorePath = data[2].Value as string ?? string.Empty;
                            return binding.IsValid;
                        }
                    }

                    // (Optional) If you ever migrate to "ProjectStore" dictionary, read it here.
                    // Keeping this block as a safe noop for now.
                }
            }
            catch
            {
                // swallow — we just report "no binding"
            }

            // Optional legacy fallback (if you still have this helper in your codebase)
            try
            {
                var tx = db.TransactionManager.StartOpenCloseTransaction();
                using (tx)
                {
                    var maybePath = ProjectTagPath.GetPath(db, tx);
                    if (!string.IsNullOrWhiteSpace(maybePath))
                    {
                        binding = new ProjectBinding
                        {
                            ProjectId = string.Empty,
                            Name = string.Empty,
                            StorePath = maybePath
                        };
                        return true;
                    }
                }
            }
            catch { /* ignore */ }

            return false;
        }

        /// <summary>Convenience overload for current MDI doc.</summary>
        public static bool TryGetBoundProject(out ProjectBinding binding)
        {
            var doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
            return TryGetBoundProject(doc, out binding);
        }

        // ------------------- Step 6: watch & notify on changes -------------------

        /// <summary>
        /// Start watching DocumentManager for activation/create/destroy to detect binding changes.
        /// Safe to call multiple times.
        /// </summary>
        public static void StartWatching()
        {
            if (_watching) return;
            var dm = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager;
            dm.DocumentActivated += OnDocumentActivated;
            dm.DocumentCreated += OnDocumentCreated;
            dm.DocumentDestroyed += OnDocumentDestroyed;

            // Seed any currently-open docs
            foreach (Document d in dm)
            {
                CacheCurrentBinding(d);
            }

            _watching = true;
        }

        private static void OnDocumentCreated(object sender, DocumentCollectionEventArgs e)
        {
            CacheCurrentBinding(e.Document);
        }

        private static void OnDocumentDestroyed(object sender, DocumentDestroyedEventArgs e)
        {
            // We no longer have the Document instance, but we can prune stale entries later.
            // For safety, clear all dead docs.
            var dm = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager;
            var live = new HashSet<Document>();
            foreach (Document d in dm) live.Add(d);

            var toRemove = new List<Document>();
            foreach (var kv in _lastStoreByDoc)
                if (!live.Contains(kv.Key)) toRemove.Add(kv.Key);

            foreach (var d in toRemove) _lastStoreByDoc.Remove(d);
        }

        private static void OnDocumentActivated(object sender, DocumentCollectionEventArgs e)
        {
            NotifyIfChanged(e.Document);
        }

        /// <summary>
        /// Public helper — call after your commands (bind/new/repair) to force a re-check.
        /// </summary>
        public static void NotifyIfChanged(Document doc)
        {
            if (doc == null) return;

            string oldPath = GetCached(doc);
            TryGetBoundProject(doc, out var binding);
            var newPath = binding.StorePath ?? string.Empty;

            if (!StringEquals(oldPath, newPath))
            {
                _lastStoreByDoc[doc] = newPath;
                RaiseProjectChanged(doc, binding, oldPath, newPath);
            }
        }

        // Cache current binding (used at startup/creation)
        private static void CacheCurrentBinding(Document doc)
        {
            if (doc == null) return;
            TryGetBoundProject(doc, out var binding);
            _lastStoreByDoc[doc] = binding.StorePath ?? string.Empty;
        }

        private static string GetCached(Document doc)
        {
            if (doc != null && _lastStoreByDoc.TryGetValue(doc, out var s)) return s ?? string.Empty;
            return string.Empty;
        }

        private static void RaiseProjectChanged(Document doc, ProjectBinding binding, string oldPath, string newPath)
        {
            try { ProjectChanged?.Invoke(null, new ProjectBindingChangedEventArgs(doc, binding, oldPath, newPath)); }
            catch { /* never propagate */ }
        }

        private static bool StringEquals(string a, string b)
            => string.Equals(a ?? string.Empty, b ?? string.Empty, StringComparison.OrdinalIgnoreCase);
    }
}

// File: AutoCAD2020/ProjectStore/BindCurrentDrawingCommand.cs
// v2.1.0 (C# 7.3)
// Binds the active DWG to a ProjectStore in the same folder and ensures
// both the CBI["Project"] identity record and the canonical
// CBI.ProjectStore.Path key (StoreGuards) are written.

using System;
using System.Collections.Generic;
using System.IO;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.EditorInput;
using AutoCAD_Shared_Project.ProjectStore;

namespace AutoCAD2020.ProjectStore
{
    public static class BindCurrentDrawingCommand
    {
        private const string SupportFolderName = "CBI_Support";
        private const string StoreFileName = "Project_Store.json";

        /// <summary>
        /// Bind the active DWG to the project store in the same folder.
        /// If store doesn't exist, create support folder + new store and add this DWG.
        /// If it exists, append this DWG if missing.
        /// Also writes the canonical StoreGuards NOD key so other tools can
        /// resolve the ProjectStore path non-interactively.
        /// </summary>
        public static void Run()
        {
            var doc = Application.DocumentManager.MdiActiveDocument;
            if (doc == null) return;
            var ed = doc.Editor;

            // Require saved drawing
            var dwgPath = doc.Name;
            if (string.IsNullOrWhiteSpace(dwgPath) || string.IsNullOrWhiteSpace(Path.GetDirectoryName(dwgPath)))
            {
                ed.WriteMessage("\nPlease save the drawing first.");
                return;
            }

            var rootDir = Path.GetDirectoryName(dwgPath);
            var supportDir = Path.Combine(rootDir, SupportFolderName);
            var storePath = Path.Combine(supportDir, StoreFileName);

            Directory.CreateDirectory(supportDir);

            var projectKey = Path.GetFileName(rootDir); // e.g. folder name
            var projectSvc = new ProjectSettingsService(storePath);

            var created = false;
            if (!File.Exists(storePath))
            {
                // New file on disk with canonical empty snapshot
                projectSvc.Write(new ProjectStoreSnapshot
                {
                    Projects = new Dictionary<string, ProjectEntry>(),
                    ActiveProjectKey = null
                });
                created = true;
            }

            // Ensure a project entry keyed by the folder name, add current DWG
            var added = projectSvc.EnsureProjectAndAddDrawing(projectKey, rootDir, dwgPath);

            // Bind NOD so the drawing “knows” which projectId it belongs to.
            try
            {
                var projectId = projectKey; // or a GUID if you prefer; keep consistent with your binder
                ProjectNodBinder.BindProjectToDrawing(doc, projectId, projectKey, storePath);
            }
            catch
            {
                // best-effort; non-fatal
            }

            // NEW: also bind the canonical ProjectStore path via StoreGuards
            try
            {
                string bindError;
                if (!StoreGuards.TryWritePath(doc.Database, storePath, out bindError) &&
                    !string.IsNullOrWhiteSpace(bindError))
                {
                    ed.WriteMessage("\n[ProjectStore] Warning: failed to bind ProjectStore path into NOD: " + bindError);
                }
            }
            catch
            {
                // keep this best-effort as well; don't hard-crash the bind command
            }

            // Feedback
            if (created)
                ed.WriteMessage($"\nCreated CBI support folder and project store:\n  {storePath}");

            ed.WriteMessage(added
                ? $"\nAdded drawing to project '{projectKey}'."
                : $"\nDrawing already listed in project '{projectKey}'.");

            AutoCAD2020.PropertiesHub.CbiPropertiesPalette.RefreshBannerIfOpen();
        }
    }
}

// -----------------------------------------------------------------------------
// Project: AutoCAD2020
// File: SheetSetTraversal.cs
// Version: 1.0.0
// Changelog:
//   1.0.0 - Traverses a Sheet Set (.dst) and produces a shared SheetSetSnapshot.
// -----------------------------------------------------------------------------

using System;
using System.IO;
using System.Runtime.InteropServices;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.EditorInput;

// Digi-Ants.AutoCAD.SheetSetManager (per your prior work)
using Autodesk.AutoCAD.Interop; // often needed for COM plumbing in some environments
using Autodesk.AutoCAD.Interop.Common;

using AutoCAD_Shared_Project.ProjectStore.SheetSetIntegration;

// If you are using Digi-Ants.AutoCAD.SheetSetManager, your namespaces may differ.
// Replace these using directives with your actual references once you confirm the exact types available:
// using DigiAnts.AutoCAD.SheetSetManager;
// using DigiAnts.AutoCAD.SheetSetManager.AcadSm;

namespace AutoCAD2020.ProjectStore.SheetSetIntegration
{
    internal static class SheetSetTraversal
    {
        /// <summary>
        /// Reads the sheet set at dstPath and produces a snapshot suitable for core sync.
        /// </summary>
        public static bool TryCaptureSnapshot(string dstPath, out SheetSetSnapshot snapshot, out string message)
        {
            snapshot = null;
            message = null;

            if (string.IsNullOrWhiteSpace(dstPath))
            {
                message = "SheetSet path is empty.";
                return false;
            }

            if (!File.Exists(dstPath))
            {
                message = "SheetSet file does not exist: " + dstPath;
                // Still create snapshot with missing file marker so conflicts can record it.
                snapshot = new SheetSetSnapshot
                {
                    SheetSetPath = dstPath,
                    SheetSetName = Path.GetFileNameWithoutExtension(dstPath),
                    CapturedUtc = DateTime.UtcNow
                };
                return true;
            }

            try
            {
                // NOTE:
                // The exact traversal depends on which SheetSetManager wrapper you have:
                // - Autodesk.AcSmSheetSetMgr COM
                // - Digi-Ants wrapper around the COM interfaces
                //
                // Because you locked "core logic in shared" and "AutoCAD refs only in AutoCAD2020",
                // we keep traversal here, but we need your *confirmed* API types to finalize.
                //
                // For now, we provide a safe placeholder that captures the basic header.
                snapshot = new SheetSetSnapshot
                {
                    SheetSetPath = dstPath,
                    SheetSetName = Path.GetFileNameWithoutExtension(dstPath),
                    CapturedUtc = DateTime.UtcNow
                };

                // TODO (requires your confirmed SSM API types):
                // - Load database via AcSmSheetSetMgr.GetDatabase(dstPath, true)
                // - Traverse IAcSmSubset hierarchy
                // - For each IAcSmSheet:
                //     - Extract Name, Number, and drawing path (via layout reference / Callout / file reference)
                // - Populate snapshot.Subsets and snapshot.Sheets
                // - Optionally mark DrawingMissing based on File.Exists for resolved DWG paths

                message = "Snapshot captured (header only). Paste your SSM traversal code/types so we can finalize traversal.";
                return true;
            }
            catch (System.Exception ex)
            {
                message = "Failed to capture SheetSet snapshot: " + ex.Message;
                return false;
            }
        }
    }
}

## Response:
12/31/2025, 3:45:45 PM

> **Revising project plan and handling updates**
>
> Okay, I need to update the plan now that I’ve got the class code. I’m not sure about the SSM traversal types since the user pasted a placeholder. Also, there’s a scope issue with AutoCAD references in shared uses—I'd like to propose moving StoreGuards to AutoCAD2020. I’ll update the file tree and flag the issues gently. Also, I need to align the binding adapter for shared and fix the missing fields in ProjectStoreV1.
>
> **Clarifying project store schema and requesting missing info**
>
> I need to clarify which ProjectStore schema is canonical. They pasted multiple models, and I can’t implement without knowing where to store the sheet set details, state, health, and conflicts. Since schema drift isn't allowed, I’ll propose adding a partial extension class but must request the correct Project_Store.json template to continue.
>
> Thought for 12s

Understood — and thank you for pasting the current models/services. Based on what you provided, there are **two critical integration realities** we must address **before** I can safely produce the end-to-end Attach/Sync implementation without violating your locked decisions (no schema drift, shared project has no AutoCAD refs).

---

## 1) Scope Guard / Architecture compliance issue detected

### Issue
You pasted `StoreGuards.cs` under **AutoCAD_Shared_Project.ProjectStore**, but it contains **AutoCAD references**:

- `Autodesk.AutoCAD.ApplicationServices`
- `Autodesk.AutoCAD.DatabaseServices`
- `Autodesk.AutoCAD.EditorInput`

That violates your locked decision:

- **“AutoCAD references only in AutoCAD2020 project”**
- **Shared project must not reference AutoCAD assemblies**

### Required action (compliant fix)
`StoreGuards.cs` must live in **AutoCAD2020** (or be split into two files: a Shared “pure” component + an AutoCAD2020 wrapper). For this thread’s Active Tool, we only need the **non-interactive resolver/binder** methods anyway.

I will not move or refactor it in-place until you tell me whether this file currently exists in Shared *in your repo*, or whether it was pasted from AutoCAD2020 by mistake.

---

## 2) Schema mismatch risk: ProjectState + Health + Conflicts are not present (in the pasted v1 model)

Your locked requirements for this tool include:

- ProjectState conflict handling: **Synced-Degraded**
- Conflict detection + persistence
- ProjectState + Health transitions

However, the provided `AutoCAD_Shared_Project.ProjectStore.V1.ProjectStoreV1` does **not** contain any of the following fields (at least not in the code you pasted):

- `ProjectState`
- `Health`
- `Conflicts` (or similar persistent conflict collection)
- `SheetSetPath` binding in the JSON model

So, if I implement persistence right now, I would be forced to **invent new fields**, which is **schema drift** and explicitly disallowed.

### Required input (to proceed correctly)
Please paste the existing class(es) that represent the **Project_Store.json schema v1.0 “full template” used by this tool**, specifically the portion where you currently store:

- SheetSet binding path (expected: something like `Config.SheetSetPath` or similar)
- ProjectState + Health fields
- Conflict persistence collection (and/or history entry format for conflicts)

If those fields exist only in JSON today and not in POCOs yet, paste the template JSON and we will generate POCOs that match it **exactly**.

---

# File tree of affected/new files (updated with what you pasted)

Below is the **actual tree impact** given the material you posted and the compliance requirement above.

## AutoCAD_Shared_Project

### Existing files you pasted (already in solution)
```
AutoCAD_Shared_Project
└─ ProjectStore
   ├─ ProjectStoreV1.cs                       (namespace AutoCAD_Shared_Project.ProjectStore.V1)
   ├─ ProjectStoreModels.cs                   (legacy registry snapshot model)
   ├─ ProjectStoreService.cs                  (Load/SaveAtomic + readers/writers)
   ├─ ProjectStoreValidator.cs
   ├─ ProjectSettingsService.cs               (legacy project registry service)
   ├─ NewProjectBuilder.cs
   └─ SafeStoreAllocator.cs
```

### Existing file pasted but **architecture-noncompliant** (AutoCAD refs in Shared)
```
AutoCAD_Shared_Project
└─ ProjectStore
   └─ StoreGuards.cs   <-- MUST be in AutoCAD2020 (or split) to honor your rules
```

### New files proposed (Sheet Set Integration core — no AutoCAD refs)
```
AutoCAD_Shared_Project
└─ ProjectStore
   └─ SheetSetIntegration
      ├─ SheetSetSnapshot.cs
      ├─ SheetSetSyncRequest.cs
      ├─ SheetSetSyncResult.cs
      ├─ SheetSetSyncService.cs
      ├─ ProjectStateMachine.cs
      ├─ ConflictModels.cs
      └─ PathNormalization.cs
```

### Affected shared files (modified)
```
AutoCAD_Shared_Project
└─ (none yet)
```

> Once you paste the real v1.0 store fields for SheetSetPath/State/Health/Conflicts, we will add **one binding adapter** file. Whether that lives in Shared or AutoCAD2020 depends on where the store model types live and what you want to keep “pure.”

---

## AutoCAD2020

### Existing files you pasted (already in solution)
```
AutoCAD2020
└─ ProjectStore
   ├─ ProjectNodBinder.cs
   ├─ ProjectBindingService.cs
   └─ BindCurrentDrawingCommand.cs
```

### New files proposed (Sheet Set Integration commands + traversal)
```
AutoCAD2020
└─ ProjectStore
   └─ SheetSetIntegration
      ├─ Command_AttachSheetSet.cs
      ├─ Command_SyncSheetSet.cs
      ├─ ProjectStoreResolver.cs              (may be replaced by StoreGuards non-interactive)
      └─ SheetSetTraversal.cs                 (real traversal implementation pending)
```

### Affected AutoCAD2020 files (modified)
```
AutoCAD2020
└─ Commands.cs
```

---

# Next step (required to continue without schema drift)

Please paste **only the existing code** (or the JSON template) that defines where these live in the v1.0 store:

1) **SheetSet path** field (the bound `.dst` path)  
2) **ProjectState** field  
3) **Health** field  
4) **Conflicts persistence** field(s) (or the history representation you want used)

Once I have that, I can produce:
- the binding adapter implementation (no guessing),
- end-to-end Attach and Sync commands,
- and persistence that follows SOP v1.0 exactly.

If you also paste your working Sheet Set traversal code (the `IAcSmSubset` hierarchy traversal you referenced previously), I will finalize `SheetSetTraversal.cs` as drop-in code immediately.

## Prompt:
12/31/2025, 4:01:12 PM

Here are some more classes.

One thing I want to add. A prior threat went off the rails and made changes to the project store that made no sence. Creating a projevtstoreV1 class but how it interacted with the core code got weird. Like you have seen discrepancy let me know because I dropped the changes and I fear the code is messy at best on its operation. I know this is outside the scope of this conversation but if we can document it for future change that would be great.

## Response:
12/31/2025, 4:01:15 PM

Noted. We can document the “went off the rails” ProjectStore inconsistencies **without changing any code** in this thread, and keep it as a future stabilization task. That stays inside Scope Guard (architecture notes only; no OPC/valve/tag allocation changes).

## Current discrepancies I can already see (based on what you pasted)

### 1) Multiple “ProjectStore” schemas coexist under the same umbrella
You currently have at least **three competing shapes**:

1) **V1 POCO store**: `AutoCAD_Shared_Project.ProjectStore.V1.ProjectStoreV1`  
   - Fields: `Version`, `Project`, `Config`, `Systems`, `Tags`, `Released`, `History`
   - This is tag-focused and validated by `ProjectStoreValidator`.

2) **Legacy registry snapshot**: `AutoCAD_Shared_Project.ProjectStore.ProjectStoreSnapshot` / `ProjectEntry`  
   - Fields: `Projects`, `ActiveProjectKey`, `Tagging`, `Blocks` etc.
   - This is effectively a different product (“Project Registry”), not the same as v1.

3) **JObject-based store**: `SafeStoreAllocator.AllocateInMemory(JObject root, ...)`  
   - Expects JSON tokens like `VersionTick`, `Tagging`, `Released` as **JObject**, and history entries as `{ WhenUtc, Who, Action, ... }`
   - This does **not** match the V1 POCO (`Released` is `List<TagReleaseRecordV1>`, not a dictionary/JObject; `HistoryEntryV1` fields differ: `Utc/Action/By/Details`)

**Impact:** code paths may silently write a shape that other paths cannot validate/deserialize cleanly, resulting in “weird interaction” and partial tolerance behaviors (your `Load()` catches validation failures and continues).

---

### 2) AutoCAD references exist in Shared (hard architecture violation)
`StoreGuards.cs` is in **AutoCAD_Shared_Project**, but references AutoCAD assemblies. That violates your locked rule:
- AutoCAD references only in AutoCAD2020 project

**Impact:** either the shared project is incorrectly referencing AutoCAD, or the file is in the wrong project. This is likely contributing to “messy operation,” because it blurs the boundary you rely on for maintainability.

---

### 3) “ProjectStoreService.ReadGlobalValvePrefix” reads multiple formats (and can hide breakage)
`ReadGlobalValvePrefix` attempts to interpret:
- legacy registry (`Projects`…`Tagging`…`Valve`…`Prefix`)
- v1 store via `Systems["VALVE"]["Prefix"]`
- older “Tags object”
- `Config.GlobalValvePrefix`

This is useful for migration, but it also makes it easy for a store file to drift into a hybrid shape without an immediate hard failure.

---

## Documentation deliverable for future stabilization (no code changes now)

I recommend we create a single technical-debt record in your GreatLibrary / Obsidian vault (or as a repo markdown file later). Here is a ready-to-paste note format.

### ProjectStore Tech Debt Note (template)
**Title:** `ProjectStore Schema Drift & Dual-Format Entanglement (Stabilization Backlog)`  
**Owner:** Brian  
**Status:** Logged / Deferred  
**Date:** 2025-12-31  
**Applies To:** `AutoCAD_Shared_Project.ProjectStore`, `AutoCAD2020.ProjectStore`

**Symptoms observed**
- POCO v1 store exists, but runtime behavior suggests multiple JSON shapes are produced/consumed.
- Core code tolerates invalid/partial store files, making drift harder to detect.
- Conflicting “ProjectStore” meanings: Tag store vs Project Registry snapshot.

**Root causes**
- Coexisting schemas:  
  - `V1.ProjectStoreV1` (tag store)  
  - `ProjectStoreSnapshot` (registry)  
  - `SafeStoreAllocator` JObject schema (v0.4 semantics)
- AutoCAD-dependent guard code placed in Shared (`StoreGuards.cs`).
- Read helpers that attempt “smart parsing” across formats hide incompatibilities.

**Risks**
- Silent corruption or hybrid store files.
- Validators bypassed in production flows.
- Future features (SheetSet integration, conflicts, health/state) cannot be added safely without a single canonical schema.

**Stabilization plan (future thread)**
1) Freeze canonical schema: confirm which JSON file is “Project_Store.json” for CBI tools.
2) Rename/segregate legacy registry code (`ProjectRegistry*`) to eliminate ambiguity.
3) Align `SafeStoreAllocator` to the canonical schema (or move it to a dedicated allocator file format).
4) Enforce strict validation in write paths; allow migration only through explicit migration steps.
5) Move `StoreGuards` AutoCAD code into AutoCAD2020 and expose a Shared interface if needed.
6) Add a `ProjectStoreSchemaInspector` diagnostic that reports detected shape + version.

**Acceptance criteria**
- Only one canonical ProjectStore schema is written going forward.
- All writers produce JSON that round-trips through the canonical POCO model.
- Any legacy formats are read-only or migrated in a single explicit process.

---

## To proceed in *this* thread (SheetSet Integration), I need the “more classes”
You said “Here are some more classes,” but they didn’t come through in your message. Paste them when ready, and I will:

1) Update the **affected/new file tree** if these classes change where we plug in.
2) Identify additional discrepancies **only as documented tech debt** (not as refactors here).
3) Continue building the SheetSet Attach + Sync implementation without schema drift.

When you paste the additional classes, include specifically anything that mentions:
- `ProjectState`, `Health`, `Conflicts`, `SheetSetPath`, `Drawings`, or `Config` extensions,
- and any “ProjectStore SOP v1.0” binding points you already implemented.

## Prompt:
3/24/2026, 3:34:01 PM

Tool Summary – Current Thread

## Response:
3/24/2026, 3:34:01 PM

Below is the **Tool Summary – Current Thread** for the **ProjectStore Sheet Set Integration** tool. This is written in the same “Architect’s Lens” style you’ve been using so it can be logged in your tool registry / GreatLibrary.

---

# Tool Summary – Current Thread  
## Tool: ProjectStore Sheet Set Integration  
**Scope:** Attach Sheet Set + Sync Sheet Set  
**Status:** Architecture + Implementation Planning (Pre-Implementation)  
**Thread Objective:** Implement Attach + Sync workflow tied to ProjectStore

---

# 1. Tool Purpose
The **ProjectStore Sheet Set Integration** tool connects a **Sheet Set (.dst)** to a **ProjectStore** and keeps the drawing list synchronized between them.

This tool establishes the relationship:

```
Drawing → ProjectStore → Sheet Set → Drawings in Project
```

This allows:
- Project health tracking
- Missing drawing detection
- Archived drawing tracking
- Project-level operations across all sheets
- Future OPC / Tag / QAQC tools to rely on ProjectStore instead of Sheet Set directly

This tool is **infrastructure**, not a user-facing feature.

---

# 2. Commands Provided by This Tool

## Command 1 — Attach Sheet Set
**Purpose:** Bind a Sheet Set to an existing ProjectStore.

### Expected Workflow
1. User runs Attach command.
2. Tool resolves ProjectStore from current drawing.
3. Tool prompts for `.dst` file.
4. Tool reads Sheet Set and extracts drawing list.
5. Tool writes SheetSetPath into ProjectStore.
6. Tool writes drawings into ProjectStore.
7. ProjectState → **Synced**
8. Health → **Healthy**
9. Store saved atomically.

### Result
ProjectStore now knows:
- Which Sheet Set belongs to the project
- Which drawings belong to the project

This command is typically run **once per project**.

---

## Command 2 — Sync Sheet Set
**Purpose:** Synchronize ProjectStore drawings with the Sheet Set.

### Expected Workflow
1. User runs Sync command.
2. Tool resolves ProjectStore.
3. Tool reads SheetSetPath from store.
4. Tool reads drawings from Sheet Set.
5. Tool compares with ProjectStore drawings.
6. Tool detects:
   - Added drawings
   - Missing drawings
   - Archived drawings
7. Tool updates ProjectStore.
8. Tool updates ProjectState and Health.
9. Tool records conflicts and history.
10. Store saved atomically.

### Result
ProjectStore stays synchronized with Sheet Set over time.

This command is typically run **periodically**.

---

# 3. ProjectState Model
The system uses a **ProjectState + Health model**.

## ProjectState
Represents synchronization state between ProjectStore and Sheet Set.

Possible states:
```
Unbound          → No Sheet Set attached
Synced           → Store and Sheet Set match
Synced-Degraded  → Sheet Set found but conflicts exist
OutOfSync        → Differences detected
Error            → Sheet Set missing or unreadable
```

## Health
Represents overall project health.
```
Healthy
Warning
Error
```

### Example State Logic
| Condition | State | Health |
|----------|-------|--------|
| No Sheet Set | Unbound | Warning |
| Fully matching | Synced | Healthy |
| Drawings missing | Synced-Degraded | Warning |
| Sheet Set missing | Error | Error |
| Differences detected | OutOfSync | Warning |

---

# 4. Drawing Status Model
Each drawing in ProjectStore has a status:

```
Active
MissingFile
Archived
```

### Meaning
| Status | Meaning |
|-------|--------|
| Active | Exists in Sheet Set and file exists |
| MissingFile | Listed but file missing |
| Archived | Previously existed but removed from Sheet Set |

This allows ProjectStore to maintain **project history**, not just current state.

---

# 5. Architecture Separation (Important)
This tool is split across **two projects**:

## AutoCAD_Shared_Project (NO AutoCAD references)
Contains:
- Sync logic
- Conflict detection
- State machine
- Drawing comparison
- Path normalization
- Models
- Sync service

This code must be pure C# and testable outside AutoCAD.

---

## AutoCAD2020 (AutoCAD references)
Contains:
- Commands
- Sheet Set traversal
- ProjectStore resolution from drawing
- Editor messages
- File dialogs
- NOD binding
- Interaction with AutoCAD API

This project acts as the **bridge** between AutoCAD and Shared logic.

---

# 6. File Responsibilities (High Level)

## Shared Project – Sync Core
Responsible for:
- Comparing Sheet Set drawings vs Store drawings
- Determining Added / Missing / Archived
- Updating ProjectState
- Updating Health
- Recording conflicts
- Producing SyncResult
- Updating ProjectStore model
- No AutoCAD calls

## AutoCAD2020 – Commands + SheetSet Access
Responsible for:
- Reading Sheet Set
- Converting Sheet Set → Snapshot
- Resolving ProjectStore path from drawing
- Loading and saving store
- Calling SyncService
- Editor output

---

# 7. Data Flow Diagram (Important)
```
User Command
     ↓
AutoCAD2020 Command
     ↓
Resolve ProjectStore Path
     ↓
Load ProjectStore JSON
     ↓
Read Sheet Set (.dst)
     ↓
Build SheetSetSnapshot
     ↓
Call SheetSetSyncService (Shared)
     ↓
Service updates store model
     ↓
Return SyncResult
     ↓
Save ProjectStore JSON
     ↓
Editor Output Summary
```

---

# 8. What This Tool Enables (Future Tools Depend On This)
This tool is foundational for:

| Future Tool | Depends On |
|-------------|------------|
| OPC Tools | ProjectStore drawing list |
| Tag allocation across drawings | ProjectStore |
| Valve association across sheets | ProjectStore |
| QA/QC across project | ProjectStore |
| Missing drawing reports | ProjectStore |
| Project health dashboard | ProjectStore |
| Batch tools | ProjectStore |
| Cross-drawing search | ProjectStore |

This tool is essentially the **project backbone**.

---

# 9. Current Thread Status
## Completed in this thread
- Scope defined
- Architecture defined
- File layout defined
- Class layout defined
- State model defined
- Drawing status model defined
- Data flow defined
- Schema drift risks identified
- ProjectStore inconsistencies documented

## Next Steps (Implementation Order)
Implementation should proceed in this order:

1. Shared Models
2. SheetSetSnapshot
3. Conflict Models
4. PathNormalization
5. ProjectStateMachine
6. SheetSetSyncService
7. AutoCAD SheetSetTraversal
8. AttachSheetSet Command
9. SyncSheetSet Command
10. Commands.cs registration
11. Testing workflow

---

# 10. One-Sentence Tool Definition (Registry Entry)
**ProjectStore Sheet Set Integration:**  
*Attaches a Sheet Set to a ProjectStore and maintains synchronized drawing membership, project state, health status, and conflict history for project-level operations.*

---

If you want, the next step should be:

**Class list and file layout with namespaces and responsibilities**, then we start writing the drop-in code in the correct order.



---
Powered by [ChatGPT Exporter](https://www.chatgptexporter.com)