# 17. Open Questions

**Status:** Draft
**Date:** 2026-08-17 (items 1-5); updated 2026-08-31 (items 6-7 added); updated 2026-09-16 (item 7 resolved, items 8-10 added)

> The points listed here were not resolved in the design discussions as of the dates above. They are not for an agent to fill in unilaterally; update this file together with the relevant document once a decision-maker has made a call.

## 1. Should an Overlay's "Purpose Label" Be Done at MVP, or Deferred?

- Background: in [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md) (a) parameter adjustment, an idea surfaced to give overlays a purpose label such as "for real-hardware reproduction" or "for training speed-up." This would help manage overlays as their number grows, but may be over-engineering at the MVP stage.
- Related document: [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md)

## 2. Recording Format for Simplification Rules

- Background: [15_Simplification_Automation_Strategy_EN.md](./15_Simplification_Automation_Strategy_EN.md) settled on the policy of turning semantic/purpose-level simplification judgments into an asset as "simplification rules," but how to express those rules and where (which file, which format) they should live in the repository is undecided.
- Related document: [15_Simplification_Automation_Strategy_EN.md](./15_Simplification_Automation_Strategy_EN.md)

## 3. A Standard for the Structured Question Posed to the Agent for Semantic/Purpose-Level Simplification

- Background: [15_Simplification_Automation_Strategy_EN.md](./15_Simplification_Automation_Strategy_EN.md) settled on the policy of "asking the agent/human in a structured way," but the standard for how to ask (prompt format, or schema) is not yet defined.
- Related document: [15_Simplification_Automation_Strategy_EN.md](./15_Simplification_Automation_Strategy_EN.md)

## 4. Minimum Spec of the State/Data Supply API (the "env face")

- Background: [12_Layering_and_ML_Boundary_EN.md](./12_Layering_and_ML_Boundary_EN.md) settled that USO exposes an env-like face (equivalent to reset/step), but the minimum spec (concrete interface signatures, how it maps to snapshots, etc.) is not yet settled.
- Candidate approach: it has been proposed to first implement a minimal replay for PyBulletFleet alone to get a feel for it, and only then settle a common schema — but this ordering itself is not yet agreed.
- Related document: [12_Layering_and_ML_Boundary_EN.md](./12_Layering_and_ML_Boundary_EN.md)

## 5. Which to Implement First: Engine-Specific Adjustment (Overlay) or the Conversion Tool

- Background: [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md) clarified the need for two distinct mechanisms — (a) overlay and (b) conversion tool — but which to implement first is undecided.
- Related document: [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md)

## 6. Exact Schema for Per-Joint Snapshot Fields

- Background: [18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md) concluded that the snapshot schema needs `joint_positions` / `joint_velocities` fields for articulated assets to remain restoration-sufficient, but the exact shape (naming convention, units, how joints are keyed for assets with many DOFs, how this interacts with delta snapshots) is not settled.
- Related document: [18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md), [03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md)

## 7. Where Should Scene-Wide "Reproduction Info" (Originally Framed as Camera, Lighting, Domain Randomization, Seed) Live? — Resolved

- Background: [18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md) identified that regenerating rendered images from simulated state (instead of storing images per step) requires additional information — camera pose/intrinsics, lighting, domain-randomization settings, random seed — that is not per-asset world state. Whether this belongs inside the snapshot format or stays entirely in an external consumer's own metadata was undecided.
- **Resolved (2026-09-16):** it lives inside the snapshot format, as its own `reproduction_info` section (global, not per-asset) holding only settings with no single owning asset — lighting, domain randomization, and the seed — kept distinct from a newly introduced general-purpose `meta_data` field. Camera pose/intrinsics were explicitly excluded from `reproduction_info` on further review: a camera is itself an asset, not scene-wide state — see [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md) (which also covers this exclusion) and [03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md).
- Related document: [18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md), [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md)

## 8. Exact Sub-Field Shape of `reproduction_info` and `meta_data`

- Background: [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md) settled that `reproduction_info` (lighting/domain-randomization/seed — camera is deliberately excluded, see that chapter) and `meta_data` (freeform) are global fields on the snapshot, distinct from per-asset state. The exact internal shape — how domain-randomization parameters are keyed, whether `meta_data` values are constrained to any type, and how partial updates to either field are expressed in a delta snapshot — is not yet settled.
- Related document: [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md), [03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md)

## 9. Should `meta_data` / `properties` Support a Registered, Typed Extension Mechanism?

- Background: [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md) noted that folding `reproduction_info` into `meta_data` only relocates the required/freeform distinction rather than removing it. This raised a further, forward-looking question: should a specific external consumer (not USO core) be able to register its own named, typed schema for data it attaches inside `meta_data` or `properties`, gaining the same kind of validation/required-field guarantees intended for `reproduction_info`'s eventual schema (its exact shape is itself still open — see Open Question 8), without USO core knowing about that schema in advance? [20_MetaData_Extensibility_Patterns_EN.md](./20_MetaData_Extensibility_Patterns_EN.md) surveys prior art (Kubernetes CRDs, Protobuf `Any`, CloudEvents, glTF extensions, OpenUSD schemas, among others) and lists candidate approaches, but does not decide. Not needed for any currently known requirement.
- Related document: [20_MetaData_Extensibility_Patterns_EN.md](./20_MetaData_Extensibility_Patterns_EN.md), [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md)

## 10. Should a Shared, Generated (Not Hand-Written) Snapshot Validation/Serialization/Replay Library Be Built, and When?

- Background: once the snapshot schema (and its extension mechanism — see Open Questions 6, 8, and 9) stabilizes, per-language libraries for schema validation, conversion to/from native data structures (e.g., a Python dict), and temporal reconstruction ("given a full snapshot plus deltas, resolve the state at time T" — the replay capability from [06_Logging_Replay_EN.md](./06_Logging_Replay_EN.md)) would let every compliant simulation node share the same logic instead of each reimplementing it. Building this now, before the schema stabilizes, risks churn and prematurely freezing decisions that are still deliberately open.
- If and when this is built, it should follow the same "define once, generate per language" pattern already committed to for gRPC in [09_Build_Strategy_EN.md](./09_Build_Strategy_EN.md) (`.proto` → `protoc` → per-language code via `scripts/gen_grpc.sh`), rather than hand-written, independently maintained libraries per language — the latter would recreate the same "duplicated logic drifting apart across languages" problem Principle 1 exists to avoid. Note that generation only goes so far: `protoc`-style tooling produces typed data bindings and (de)serialization code, not the snapshot merge/replay algorithm or richer validation constraints (e.g., "both joint fields present together," per Open Question 6) — those still need a deliberately shared implementation of their own, generated or not, so they are not left to drift the same way.
- A proposed implementation path (not yet agreed, but consistent with the bottom-up approach in [16_Current_Status_and_Rollout_Approach_EN.md](./16_Current_Status_and_Rollout_Approach_EN.md)): implement first within PyBulletFleet alone (referencing this repository's design), then extract the common logic out of that concrete implementation into a reusable library, then generalize it for other simulators — abstracting from multiple concrete instances rather than designing the shared library up front.
- Related document: [06_Logging_Replay_EN.md](./06_Logging_Replay_EN.md), [09_Build_Strategy_EN.md](./09_Build_Strategy_EN.md), [16_Current_Status_and_Rollout_Approach_EN.md](./16_Current_Status_and_Rollout_Approach_EN.md), [20_MetaData_Extensibility_Patterns_EN.md](./20_MetaData_Extensibility_Patterns_EN.md)

## Related Documents

- [11_Design_Principles_EN.md](./11_Design_Principles_EN.md)
- [12_Layering_and_ML_Boundary_EN.md](./12_Layering_and_ML_Boundary_EN.md)
- [13_Reuse_Layers_and_Format_Selection_EN.md](./13_Reuse_Layers_and_Format_Selection_EN.md)
- [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md)
- [15_Simplification_Automation_Strategy_EN.md](./15_Simplification_Automation_Strategy_EN.md)
- [16_Current_Status_and_Rollout_Approach_EN.md](./16_Current_Status_and_Rollout_Approach_EN.md)
- [18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md)
- [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md)
- [20_MetaData_Extensibility_Patterns_EN.md](./20_MetaData_Extensibility_Patterns_EN.md)
