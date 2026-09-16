# 16. USO's Connection to Existing Apps and Specs (Current Positioning)

**Status:** Draft
**Date:** 2026-08-17

## Existing Apps

- Arm (Isaac Sim)
- PyBulletFleet (kinematics / fleet)
- Planned addition: AGV (Isaac Sim or MuJoCo)

## Approach: Bottom-Up

- Build two or three concrete apps first, then abstract the unifying layer (USO's common layer) from them.
- Abstraction requires multiple concrete examples. Abstracting from a single app makes it impossible to tell whether something is that app's own concern or genuinely essential (a judgment based on Principle 1 in [11_Design_Principles_EN.md](./11_Design_Principles_EN.md)).

## Relationship With the Existing USD Loader Spec (PyBulletFleet Side)

The existing USD loader spec on the PyBulletFleet side ([`docs/design/usd-behavior-tree/spec.md`](https://github.com/yuokamoto/PyBulletFleet/blob/main/docs/design/usd-behavior-tree/spec.md) in the PyBulletFleet repository, implemented in `pybullet_fleet/usd_loader.py`) has decided:
- Read the environment via USD — an optional OpenUSD static-world importer that converts supported meshes into static `SimObject`s
- Treat prim paths as stable IDs — "Prim paths, not PyBullet body IDs, are the stable external source IDs" (per the spec)
- Normalize the conversion exactly once — every imported pose and mesh scale is normalized to PyBulletFleet's Z-up, metres convention on import, not re-derived later

This decision correctly prepares the "stable reference" that later snapshot/replay work ([03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md), [06_Logging_Replay_EN.md](./06_Logging_Replay_EN.md)) and USO's conversion tool ([14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md)) assume.

## Inventory Approach

Inventory each app along five dimensions, and classify each element as "essential" or "simulator-specific":

1. State
2. Reset
3. Assets
4. Time
5. Representation

Elements judged "simulator-specific" are further split according to the classification in [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md):

- (a) Parameter adjustment → overlay
- (b) Structural difference → conversion tool

## Related Documents

- [11_Design_Principles_EN.md](./11_Design_Principles_EN.md) — The principle underlying the bottom-up approach
- [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md) — The (a)/(b) classification applied after inventory
- [03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md), [06_Logging_Replay_EN.md](./06_Logging_Replay_EN.md) — Existing specs assumed by the USD loader spec's decisions
- PyBulletFleet repository, [`docs/design/usd-behavior-tree/spec.md`](https://github.com/yuokamoto/PyBulletFleet/blob/main/docs/design/usd-behavior-tree/spec.md) — The USD loader spec referenced above
- PyBulletFleet repository `docs/design/unified-spawn/spec.md` — Existing spec for unified YAML entity spawning/registration (a related loader/spawn spec)
