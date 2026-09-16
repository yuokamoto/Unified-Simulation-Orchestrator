# 11. Design Principles Behind USO

**Status:** Draft
**Date:** 2026-08-17

> This document records conclusions reached in a design discussion; it does not invent new design decisions at this point. Unresolved points are collected in [17_Open_Questions_EN.md](./17_Open_Questions_EN.md).

## The Problem to Solve

Every time the simulator or the robot changes, things that should conceptually stay the same — environment layout, robot structure, task logic, recovery logic — end up being rebuilt from scratch. This is real, observed waste, and removing this **"rebuilding that has nothing to do with the essence of the thing"** is the purpose of USO.

## Principle 1: Separate things that change for different reasons, and bridge the boundary with a common format

- When an element changes across simulators or robots, the reason it changes is not always the same. Sometimes it changes because of a genuine property of the task or robot; sometimes it changes only because of a given simulator's own concerns (friction model, renderer, how DOFs are represented, etc.).
- If these two are not separated, adding a new simulator drags the essential part along for a rebuild every time.
- USO therefore commits to **structurally separating things that change for different reasons, and bridging that boundary with common formats (OpenUSD, URDF/SDF, Behavior Tree XML, snapshots)**.
- This principle is the baseline that runs through nearly every chapter in this document set ([12](./12_Layering_and_ML_Boundary_EN.md) through [16](./16_Current_Status_and_Rollout_Approach_EN.md)); when in doubt about a specific design choice, return here.

## Principle 2 (derived): The source of truth is the most information-rich representation; derivation only flows toward less information

- When multiple representations point at the same object (a detailed articulated structure vs. a simplified kinematic representation, full physics vs. logic-only, etc.), it must be decided which one is authoritative.
- Converting from a richer representation to a poorer one (detailed → simplified) discards information and is therefore always possible. The reverse (simplified → detailed) cannot reconstruct lost information and is therefore impossible in principle.
- Hence, **the most information-rich representation is treated as the source of truth, and derivation flows only in the direction of reducing information.**
- This principle is concretely applied in format selection ([13](./13_Reuse_Layers_and_Format_Selection_EN.md)), handling of engine-specific adjustments ([14](./14_Engine_Specific_Adjustment_EN.md)), and the simplification automation strategy ([15](./15_Simplification_Automation_Strategy_EN.md)) — it is the other axis running through this entire document set.

## Why These Two Principles Are the Starting Point (Rationale)

- Without "separation by reason for change," adding an engine breaks the essence (task logic, robot structure, environment layout) every time. This is exactly the problem USO exists to solve.
- Without "one-directionality of the source of truth," it becomes ambiguous which representation should be maintained as authoritative, and the detailed and simplified versions drift into mutual contradiction. Making the direction explicit automatically determines how conversion tools should be designed (which direction the conversion should be written in).

## Related Documents

- [12_Layering_and_ML_Boundary_EN.md](./12_Layering_and_ML_Boundary_EN.md) — Layer/responsibility separation and the relationship with the ML ecosystem
- [13_Reuse_Layers_and_Format_Selection_EN.md](./13_Reuse_Layers_and_Format_Selection_EN.md) — Reuse potential by layer and format selection
- [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md) — Handling engine-specific adjustments (overlay / conversion tool)
- [15_Simplification_Automation_Strategy_EN.md](./15_Simplification_Automation_Strategy_EN.md) — Automation strategy for simplification
- [16_Current_Status_and_Rollout_Approach_EN.md](./16_Current_Status_and_Rollout_Approach_EN.md) — Connection to existing apps/specs
- [17_Open_Questions_EN.md](./17_Open_Questions_EN.md) — Unresolved points
- [18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md) — Applying Principle 2 to state vs. rendered image, for an external consumer of the snapshot format
- [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md) — Applying Principle 1 to split per-asset state, reproduction info, and freeform meta data into distinct snapshot fields
- [20_MetaData_Extensibility_Patterns_EN.md](./20_MetaData_Extensibility_Patterns_EN.md) — Prior art and candidate approaches for a future typed-extension mechanism within `meta_data`/`properties` (exploration, not decided)
- [02_Architecture_EN.md](./02_Architecture_EN.md) — The layered structure where this principle is implemented
