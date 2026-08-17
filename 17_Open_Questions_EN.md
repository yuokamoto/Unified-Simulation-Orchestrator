# 17. Open Questions

**Status:** Draft
**Date:** 2026-08-17

> The points listed here were not resolved in the design discussion as of 2026-08-17. They are not for an agent to fill in unilaterally; update this file together with the relevant document once a decision-maker has made a call.

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

## Related Documents

- [11_Design_Principles_EN.md](./11_Design_Principles_EN.md)
- [12_Layering_and_ML_Boundary_EN.md](./12_Layering_and_ML_Boundary_EN.md)
- [13_Reuse_Layers_and_Format_Selection_EN.md](./13_Reuse_Layers_and_Format_Selection_EN.md)
- [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md)
- [15_Simplification_Automation_Strategy_EN.md](./15_Simplification_Automation_Strategy_EN.md)
- [16_Current_Status_and_Rollout_Approach_EN.md](./16_Current_Status_and_Rollout_Approach_EN.md)
