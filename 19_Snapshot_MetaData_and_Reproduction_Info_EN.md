# 19. Snapshot Global Fields: Reproduction Info and Meta Data

**Status:** Draft
**Date:** 2026-09-16

> This document records the conclusion of a design discussion that resolves Open Question 7 in [17_Open_Questions_EN.md](./17_Open_Questions_EN.md), first raised in [18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md): where should "reproduction info" (camera, lighting, domain randomization, seed) live? It also settles a closely related question raised in the same discussion: how should genuinely user-arbitrary, non-essential data be attached to a snapshot? Unresolved points from this discussion are added to [17_Open_Questions_EN.md](./17_Open_Questions_EN.md).

## The Question

[18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md) identified that an external consumer, wanting to regenerate rendered images from simulated state rather than store an image per step, needs additional information beyond per-asset world state: camera pose/intrinsics, lighting, domain-randomization settings, and the random seed used ("reproduction info"). It left open whether this belongs inside the snapshot format or stays entirely outside it as the consumer's own metadata.

A related, initially separate question surfaced in parallel: dynamic parameters that don't fit any existing typed field (per-asset or global) also need somewhere to live — should a generic, user-defined field be added for this?

## Decision

The snapshot's `world` object gains two new fields, siblings to `assets` (schema in [03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md)):

1. **`reproduction_info`** (global, structured) — scene-wide rendering/reproduction settings that are not tied to any single object: lighting, domain-randomization settings, and the random seed. Required only for the specific purpose of regenerating a rendered image from state without storing the image. Episode-scoped and normally static for the duration of a run; carried in the full snapshot. Camera pose/intrinsics are deliberately excluded — see "Cameras Are Assets, Not Global State" below.
2. **`meta_data`** (global, freeform) — user-defined data not required for any restoration purpose. The global-scope counterpart to the existing per-asset `properties` field ([03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md)).

Per-asset arbitrary data continues to use the existing `properties` field; no new per-asset field was added, since `properties` already serves that role.

## Why `reproduction_info` Is Not Folded Into `properties` / `meta_data`

The initial framing — "isn't everything in a snapshot there because it's needed for reproduction?" — conflates two distinct senses of "reproduction":

1. **Restoration-sufficient for continuing the simulation**: pose, velocity, joint state, status, connections. This is what the bulk of the existing snapshot schema, including the joint-state extension in [18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md), already provides — required regardless of whether images are ever regenerated.
2. **Sufficient to regenerate a rendered image from that state**: camera/lighting/domain-randomization/seed. This is a narrower, additional requirement that matters only when an external consumer chooses not to store images and wants to reconstruct them later.

Only sense (2) is what `reproduction_info` covers, and only a small, fixed set of fields falls under it — not "most of the snapshot," as first appeared. Because this is a specific, structurally required set of fields for one particular feature (not an optional convenience), it must stay distinguishable from `meta_data`, which is deliberately unconstrained and carries no guarantee of being interpretable by anything else. Folding `reproduction_info` into `properties`/`meta_data` would erase this distinction, making it impossible to tell, from the schema alone, which fields a consumer can actually rely on.

## Cameras Are Assets, Not Global State

An earlier draft of `reproduction_info` included a `camera` sub-field (pose + intrinsics). Revisiting this: a camera used for observation is not itself a piece of scene-wide configuration — it is a physical thing with a pose, exactly like any other asset. Two cases:

- **Rigidly mounted sensor** (e.g., a robot's wrist camera): its world pose is always derivable from the parent asset's already-tracked state — its root pose and, for an articulated asset, its already-tracked `joint_positions` — combined with the fixed mounting offset and kinematic chain defined in the asset's model (URDF/USD). No additional snapshot field is needed: for an articulated mount the derivation runs through the full joint chain, not just a single offset from the root, but it remains entirely a function of state the snapshot already carries, so duplicating a separate pose in `reproduction_info` would just be redundant state that could drift out of sync with the parent.
- **Free-standing observation camera** (e.g., a fixed overview camera watching a workspace): this is simply another asset (`type: "camera"`) with its own `position`/`orientation`, tracked through the existing per-asset mechanism — no different from a robot or a pallet.

In both cases, intrinsics (focal length, resolution, field of view, etc.) are static properties of the camera's model, not per-step dynamic data, so they belong with the asset's static description rather than in a dynamic global field. The exact contract for expressing that static calibration (where it lives, in what shape) is not yet defined — see Open Question 11.

This leaves `reproduction_info` holding only settings that genuinely have no single owning asset — lighting and domain randomization — plus the seed. It is also a clean instance of Principle 2 in [11_Design_Principles_EN.md](./11_Design_Principles_EN.md): for a rigidly mounted sensor, the parent asset's tracked pose is the source of truth, and the camera's pose is a derived quantity that must not be duplicated as independent state.

## Relation to Principle 1

This three-way split is a direct application of Principle 1 in [11_Design_Principles_EN.md](./11_Design_Principles_EN.md) (separate things that change for different reasons, bridge with a common format):

- Per-asset dynamic state (`position`, `velocity`, `joint_positions`) changes every simulation step, for physics/logic reasons. The per-asset `properties` field is scoped the same way (per asset) but, being arbitrary user-defined data, carries no cadence guarantee of its own — it may change every step, rarely, or never.
- `reproduction_info` changes at most once per episode (typically at reset), for rendering/observer-setup reasons.
- `meta_data` changes at whatever rate a user chooses, for reasons entirely outside the framework's concern.

Keeping these as three distinct, separately named fields — rather than merging any of them — preserves the ability to reason about, and depend on, each independently.

## Related Documents

- [03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md) — Schema updated with the `reproduction_info` and `meta_data` global fields
- [11_Design_Principles_EN.md](./11_Design_Principles_EN.md) — Principle 1, applied here
- [18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md) — Where the reproduction-info requirement was first surfaced
- [17_Open_Questions_EN.md](./17_Open_Questions_EN.md) — Open Question 7 resolved here; exact sub-field shapes left as Open Question 8
