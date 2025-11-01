# Unity Essentials

This module is part of the Unity Essentials ecosystem and follows the same lightweight, editor-first approach.
Unity Essentials is a lightweight, modular set of editor utilities and helpers that streamline Unity development. It focuses on clean, dependency-free tools that work well together.

All utilities are under the `UnityEssentials` namespace.

```csharp
using UnityEssentials;
```

## Installation

Install the Unity Essentials entry package via Unity's Package Manager, then install modules from the Tools menu.

- Add the entry package (via Git URL)
    - Window → Package Manager
    - "+" → "Add package from git URL…"
    - Paste: `https://github.com/CanTalat-Yakan/UnityEssentials.git`

- Install or update Unity Essentials packages
    - Tools → Install & Update UnityEssentials
    - Install all or select individual modules; run again anytime to update

---

# Field Binder

> Quick overview: One‑shot field/property value sync between two objects via reflection. Choose A and B targets, pick members from dropdowns (with optional nested paths), and copy values A→B, B→A, or Two‑Way on scene load or on demand.

A small, reflection‑based utility component that helps you keep two values in sync. It discovers accessible fields (and optionally properties), supports nested paths up to 3 levels, validates that types match exactly, and applies the binding once (not continuously) when the scene loads or when you call Apply.

![screenshot](Documentation/Screenshot.png)

## Features
- Simple bindings between two objects
  - Pick Source A and Source B, then select member names from dropdowns
  - Binding directions: `OneWayAB`, `OneWayBA`, `TwoWay`
- Nested member paths
  - Auto‑discovers public instance fields; optionally include non‑public fields and properties
  - Supports nested access via dot paths (e.g., `Transform.localScale.x`) up to depth 3
- One‑shot application
  - Applies once After Scene Load, and can be invoked manually via `ApplyBinding()`
  - Two‑way chooses the first non‑null unequal value to copy
- Inspector feedback
  - Live validation with warnings when types are unresolved or mismatched
  - Context menu toggle to reveal more references (non‑public + properties)
- Edit mode friendly
  - Runs in Edit Mode due to `[ExecuteAlways]` for discovery and validation

## Requirements
- Unity 6000.0+
- Attribute helpers from Unity Essentials (for the inspector UI):
  - Enum Drawer: `Unity.Editor.Drawer.Enum` (provides `[Enum]` value dropdown)
  - On Value Changed: `Unity.Editor.Attributes.OnValueChanged`
  - Info: `Unity.Editor.Attributes.Info`

## Usage
1) Add the `FieldBinder` component to a GameObject
2) Set `Direction` to `OneWayAB`, `OneWayBA`, or `TwoWay`
3) Assign `SourceA` and `SourceB`
4) Choose `ReferenceA` and `ReferenceB` from the dropdowns
   - By default, you’ll see public instance fields (including nested up to 3 levels)
   - Use the component’s context menu → “Show All References” to include non‑public fields and properties
5) Enter Play Mode, or call `ApplyBinding()` to copy values according to the selected direction

Programmatic setup
```csharp
var binder = gameObject.AddComponent<FieldBinder>();
binder.Direction = FieldBinder.BindingDirection.TwoWay;
binder.SourceA = sourceA; binder.ReferenceA = "Data.Value";
binder.SourceB = sourceB; binder.ReferenceB = "Data.Value";

// Apply once now (one‑shot)
binder.ApplyBinding();

// Or apply all in scene after load (called automatically at runtime)
// FieldBinder.InitializeBindings();
```

## How It Works
- Discovery
  - Builds a base‑to‑derived type chain (excluding `MonoBehaviour`) and collects member names
  - Default: public instance fields only; “Show All References” also includes non‑public fields and readable properties
  - Nested members are expanded recursively up to depth 3 and emitted as dot paths
- Type resolution and validation
  - Resolves the final type for each dot path; disallows `string.Length` and indexers
  - Shows a warning if types cannot be resolved or don’t match exactly
- Applying a binding (one‑shot)
  - A→B: reads A and writes into B if the value type matches
  - B→A: reads B and writes into A
  - Two‑Way: if A ≠ B, prefers copying the first non‑null unequal value; otherwise copies the other direction
  - Fields and properties are set if writable (no indexers); exact type match is required
- Lifecycle
  - `[ExecuteAlways]` allows editor discovery/validation
  - On runtime After Scene Load, `InitializeBindings()` applies all registered binders once

## Notes and Limitations
- Not a live/continuous binding; it’s applied once per call. Call `ApplyBinding()` again when values change
- Exact type match is required; no implicit or numeric conversions are performed
- Indexers are not supported; collections/arrays/generics are skipped in nested discovery
- Nested traversal depth is limited to 3; UnityEngine.Object‑derived types are not traversed into
- Properties require a getter for discovery and a setter to assign; private members show up only with “Show All References” enabled
- Certain properties like `string.Length` are explicitly excluded

## Files in This Package
- `Runtime/FieldBinder.cs` – Component, reflection discovery, validation, and one‑shot apply
- `Runtime/UnityEssentials.FieldBinder.asmdef` – Runtime assembly definition

## Tags
unity, tools, binding, reflection, fields, properties, inspector, editor, workflow, utilities
