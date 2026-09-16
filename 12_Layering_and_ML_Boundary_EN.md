# 12. Layering, Responsibility Separation, and the Relationship with the Existing ML Ecosystem

**Status:** Draft
**Date:** 2026-08-17

> This document records the conclusion of applying Principle 1 from [11_Design_Principles_EN.md](./11_Design_Principles_EN.md) (separate things that change for different reasons) to the boundary between USO and the ML training pipeline.

## Scope of USO's Responsibility

USO commits to being **"orchestration of simulation execution"** only. The ML training pipeline (reward design, gradient computation, the training loop itself) sits **outside and above** USO — USO does not know about these.

## The Boundary Between Training and USO: an API That Supplies State/Data

- The boundary between USO and a training pipeline is defined as **an API that supplies state/data**.
- USO exposes an env-like face (functionality equivalent to `reset` / `step`):
  - State reset / injection
  - Randomization of initial placement
  - Parallel execution
  - Retrieval of observations

- **Important:** this functionality is not ML-only. Non-ML use cases — resetting to reproduce a bug, initializing a scenario — need the exact same functionality. USO therefore holds these not as an "ML-only API" but as **general-purpose state management functionality**.

## Why This Boundary (Rationale)

- Separation of concerns (an application of Principle 1 in [11](./11_Design_Principles_EN.md)): we don't want to touch USO when the training method (algorithm, reward design) changes, and conversely we don't want to touch the training side's code when a simulation engine is added or changed.
- USO must stay **lightweight** enough to be usable for non-ML purposes (fleet validation, sales demos, marketing). Bringing knowledge of the training loop into USO would make it heavier and more complex for these use cases.

## Relationship with the Existing ML Ecosystem (Not a Competitor)

- The ML ecosystem (RL env standards, etc.) has already matured "switching physics simulators for training." USO **does not reinvent this**.
- USO's differentiation is not "depth" but "breadth": USO's value is being a reuse layer for assets, scenarios, and state that **cuts across** fleet validation, sales, marketing, and ML — not just training.
- Toward ML, USO **conforms to** existing env standards, so it can ride directly on top of existing training frameworks (USO cooperates rather than competes).
- **Note on framing:** USO's value proposition to ML teams is better described as "cross-use-case asset/scenario reuse" rather than "simulator switching" — the latter framing is more easily mistaken as competing with existing training tools.

## Related Documents

- [11_Design_Principles_EN.md](./11_Design_Principles_EN.md) — The underlying principle for this chapter
- [02_Architecture_EN.md](./02_Architecture_EN.md) — Existing definition of the Interface Layer (Master ⇔ Node, external communication)
- [07_Interface_Messaging_EN.md](./07_Interface_Messaging_EN.md) — Existing external interface specification
- [17_Open_Questions_EN.md](./17_Open_Questions_EN.md) — The minimal spec of the state/data supply API is not yet settled (see Open Questions)
- [18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md) — A concrete external consumer sitting behind this boundary, and what it requires of the snapshot format
