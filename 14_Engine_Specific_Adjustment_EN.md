# 14. Handling Simulator-Specific Adjustments

**Status:** Draft
**Date:** 2026-08-17

> This is the core question that determines how usable USO is. On the premise that engine-specific adjustment always happens, the question is not "can it be eliminated?" but **"how do we separate it without polluting the essence?"**

## Premise

Engine-specific adjustments (differences in friction coefficients, sensor placement, degree of simplification, or even the granularity of representation itself) will always occur, USO or not. The question to ask is therefore not "can engine-specific adjustment be eliminated?" but **"how do we separate engine-specific adjustment without polluting the essence (the source of truth)?"**

Engine-specific adjustments split into two kinds by nature.

## (a) Parameter Adjustment → Separate via Overlay

- Scope: **differences in value** — friction coefficients, sensor placement, degree of simplification, and the like.
- Handling: separate as an **overlay (a diff against the essence)**. The essence (source of truth) itself is not rewritten; a diff is layered on top of it, and at runtime "essence + engine-specific overlay" is composed.
- Format-level backing: USD's sublayer/variant mechanism directly supports this "runtime composition of essence + overlay" at the format level.
- A possible extension (needs discussion): giving an overlay a "purpose label" (e.g., "for real-hardware reproduction" vs. "for training speed-up") would keep overlays manageable as their number grows. Whether to do this at MVP time may be over-engineering, so it is left as an **Open Question** (see [17_Open_Questions_EN.md](./17_Open_Questions_EN.md)).

## (b) Structural Difference → Separate via a Conversion Tool, One-Directionality Assumed

- Scope: cases where **the granularity of representation itself differs** — e.g., a detailed articulated robot vs. a kinematic point. An overlay cannot absorb this, because the difference is not a value diff but a difference in structure itself.
- Handling: a **conversion tool** derives the per-engine representation from the source of truth.
- **One-directionality is assumed**: as a direct application of Principle 2 in [11_Design_Principles_EN.md](./11_Design_Principles_EN.md), converting detailed → simplified discards information and is therefore possible, while simplified → detailed cannot reconstruct lost information and is therefore impossible. Hence **the detailed version must be kept as the source of truth**.
- Concrete example: the one-directional URDF → USD conversion described in [13_Reuse_Layers_and_Format_Selection_EN.md](./13_Reuse_Layers_and_Format_Selection_EN.md) is one instance of case (b).
- How far the "detailed → simplified" conversion itself can be automated is covered in [15_Simplification_Automation_Strategy_EN.md](./15_Simplification_Automation_Strategy_EN.md).

## Why This Two-Way Split Is Core (Rationale)

- (a) and (b) need fundamentally different machinery. (a) can be solved by "composing a diff," while (b) requires "producing distinct structures" and cannot be handled by an overlay mechanism alone. Blurring this distinction leads either to over-engineering — trying to handle a mere parameter difference with a conversion tool — or to breakage — trying to force a structural difference into an overlay.
- USO's usability hinges on whether, every time an engine-specific adjustment arises, a developer can quickly judge "is this (a) or (b)?" and route it to the right mechanism (overlay or conversion tool).

## Related Documents

- [11_Design_Principles_EN.md](./11_Design_Principles_EN.md) — The one-directionality principle
- [13_Reuse_Layers_and_Format_Selection_EN.md](./13_Reuse_Layers_and_Format_Selection_EN.md) — URDF → USD as a concrete instance of (b)
- [15_Simplification_Automation_Strategy_EN.md](./15_Simplification_Automation_Strategy_EN.md) — How far to automate the (b) conversion
- [17_Open_Questions_EN.md](./17_Open_Questions_EN.md) — Overlay purpose labels; which of (a)/(b) to implement first
