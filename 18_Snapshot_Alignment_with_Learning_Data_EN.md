# 18. Alignment Between the Snapshot Format and an External Learning-Data Recording Format

**Status:** Draft
**Date:** 2026-08-31

> This document records conclusions from a later design discussion that extends [12_Layering_and_ML_Boundary_EN.md](./12_Layering_and_ML_Boundary_EN.md) and [13_Reuse_Layers_and_Format_Selection_EN.md](./13_Reuse_Layers_and_Format_Selection_EN.md). It does not invent new design decisions; unresolved points are added to [17_Open_Questions_EN.md](./17_Open_Questions_EN.md).

## Context: A Second, Independent Consumer of the Snapshot Format

A separate teleoperation data recording effort (for imitation-learning-style training, outside this repository) needs an "observation" representation of the simulated world at each recorded step. That effort has deliberately chosen to define its observation as **the same thing as USO's SimulationSnapshot**, rather than inventing a parallel state representation.

This matters for USO because it means the snapshot format is no longer only USO's own internal concern (initialization, sync, replay — [03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md)); it is also being relied on as the **source-of-truth state format for an external consumer**. Any change to the snapshot schema now has a second stakeholder.

## Requirement Surfaced: "Restoration-Sufficient" State Needs Joint-Level Fields

The external effort requires its observation to be **restoration-sufficient**: enough to resume the simulation from that recorded point and get the same continuation. Concretely, for an articulated robot this means pose **and velocity** (pose alone cannot reconstruct in-progress motion) **and joint angles/joint velocities** — not just the rigid-body pose of the asset as a whole.

The current snapshot schema example in [03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md) only shows rigid-body fields (`position`, `orientation`, `linear_velocity`, `angular_velocity`) per asset. It does not show a place for an articulated asset's per-joint state. This is a concrete gap surfaced by the external use case, not a design flaw in isolation — USO itself has not yet needed to replay an articulated robot's internal joint state, so it was not exercised before.

**Conclusion:** the full/delta snapshot schema should be extended with per-asset joint state (joint positions and joint velocities) for articulated assets, so the snapshot remains restoration-sufficient for robots with internal degrees of freedom, not only for their root pose. The exact field shape is left open (see [17_Open_Questions_EN.md](./17_Open_Questions_EN.md)); this document only records that the gap exists and that the snapshot — not a separate format — is the right place to close it.

This is a direct instance of Principle 2 in [11_Design_Principles_EN.md](./11_Design_Principles_EN.md): simulated state is the most information-rich source of truth, and other representations (e.g., rendered images) are derived from it, not the other way around.

## The Existing Full/Delta Split Already Fits This Use Case

The external format separately needs "static structure" (what asset exists, its type/model) versus "dynamic state" (pose/velocity/joint state at a given step). This maps directly onto the existing full-vs-delta snapshot split ([03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md)): static structure is naturally carried once in the initial full snapshot, and subsequent deltas carry only what changes. No new mechanism is needed here — this is recorded as confirmation that the existing design already generalizes to this external use case, not as a new decision.

## Reproduction Info (Camera, Lighting, Seed) Is a Separate Concern

For simulated data, the external effort wants to avoid storing rendered images for every recorded step and instead regenerate them later from state, on the reasoning that in simulation the state is the source of truth and the image is a derived, reproducible artifact. Doing so requires retaining additional information beyond per-asset world state: camera pose/intrinsics, lighting, domain-randomization settings, and the random seed used.

This "reproduction info" is not part of the world state itself (it describes how an observer/renderer would reconstruct an image from that state, not the state of any simulated asset). **Resolved in a later discussion:** it is added to the snapshot format, as its own `reproduction_info` section (global, not per-asset), kept distinct from a general-purpose `meta_data` field introduced at the same time — see [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md) for the resolution and its rationale.

## Related Documents

- [03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md) — The snapshot schema this chapter proposes extending
- [11_Design_Principles_EN.md](./11_Design_Principles_EN.md) — Principle 2 (source of truth flows detail → simplified), applied here to state vs. rendered image
- [12_Layering_and_ML_Boundary_EN.md](./12_Layering_and_ML_Boundary_EN.md) — The state/data supply API boundary this external consumer sits behind
- [13_Reuse_Layers_and_Format_Selection_EN.md](./13_Reuse_Layers_and_Format_Selection_EN.md) — Format selection for the state layer
- [17_Open_Questions_EN.md](./17_Open_Questions_EN.md) — Open questions added by this chapter
