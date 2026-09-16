# 13. Reuse Layers and Format Selection

**Status:** Draft
**Date:** 2026-08-17

> This document records the conclusion of applying the two principles from [11_Design_Principles_EN.md](./11_Design_Principles_EN.md) (separation by reason for change / one-directionality of the source of truth) to the concrete questions of *what* is reused and *in which format*.

## Layering of What Can Be Reused

How well reuse carries across engines differs by layer.

| Layer | Reuse Scope | Notes |
|---|---|---|
| Assets (USD/URDF) | Across all engines | Shareable most broadly |
| Behavior Tree (task logic, recovery flows) | Shared regardless of whether physics is present | Leaf nodes (low-level actions) are engine-dependent |
| Perception model (image → state) | Only engines with rendering | Depends on training conditions |
| Policy model (state → action) | Only engines with physics | Meaningless in SimPy |

The lower the layer (assets), the broader the reuse scope; the closer to the upper layer (policy model), the more reuse scope narrows depending on engine properties (presence of physics, presence of rendering).

## Format Selection (Choose the Most Cross-Cutting Format per Asset Type)

| Target | Format | Rationale |
|---|---|---|
| Environment / scene | OpenUSD | Rich Isaac asset ecosystem, has overlay capability |
| Robot structure | URDF/SDF | Cross-robotics de facto standard; readable by PyBullet/MuJoCo/Gazebo |
| Task logic | Behavior Tree (BehaviorTree.CPP v4 XML as canonical) | Common format with proven GUI editing and C++/Python conversion (see [04_BehaviorTree_Specification](./04_BehaviorTree_Specification_EN.md)) |
| State | Snapshot (full / delta) | Unifies initialization, synchronization, bug reproduction, and replay (see [03_Snapshot_Specification](./03_Snapshot_Specification_EN.md)) |

## Important Implication: One-Directionality of URDF → USD (an Isaac Sim Constraint)

- In Isaac Sim, even robots are natively represented in USD; URDF is converted to USD on import. This conversion is **one-directional** (URDF → USD); the reverse (USD → URDF) generally does not hold.
- This is a concrete instance, in the context of format selection, of Principle 2 from [11_Design_Principles_EN.md](./11_Design_Principles_EN.md) (the source of truth is the more information-rich side; derivation flows only toward less information).
- If robot definitions need to be shared **bidirectionally** across multiple engines, URDF must be kept as the source of truth, and Isaac-specific enrichment (materials, particle effects, or other information used only on the Isaac side) must not be mixed into the URDF — it must be **separated into a different layer (a USD overlay)**.
- This overlay-based separation is itself an instance of "(a) parameter adjustment," which is handled more generally in [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md).

## Related Documents

- [11_Design_Principles_EN.md](./11_Design_Principles_EN.md) — The two principles applied in this chapter
- [03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md) — Existing specification of the state format (snapshot)
- [04_BehaviorTree_Specification_EN.md](./04_BehaviorTree_Specification_EN.md) — Existing specification of the task logic format (BT)
- [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md) — Separation of parameter adjustment via overlay
- [16_Current_Status_and_Rollout_Approach_EN.md](./16_Current_Status_and_Rollout_Approach_EN.md) — Existing decisions on PyBulletFleet's USD/environment loader
- [18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md) — Extending the state format to stay restoration-sufficient for an external consumer
