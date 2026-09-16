# 15. Automation Strategy for Simplification (Detailed → Simplified)

**Status:** Draft
**Date:** 2026-08-17

> This document covers how far the (b) structural-difference conversion (detailed → simplified) from [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md) can be automated.

## Premise

Structural simplification (detailed → simplified) is an area where automation can be pursued, but it cannot be automated uniformly. **How much can be automated differs by level**, so handling is split by level.

## Geometry/Kinematics Level → Aim for Rule-Based Automation

- Scope: convex hull computation, bounding-box conversion, collision-shape simplification, and the like.
- These are geometrically/mathematically self-contained operations — mechanical processing that does not depend on use case — so we **aim for rule-based automation** here.

## Semantic/Purpose Level → Cannot Be Decided by Geometry Alone; Ask the Agent/Human in a Structured Way

- Scope: the judgment of which degrees of freedom or elements to keep. This cannot be decided from geometric information alone — it **depends on the use case**.
- Example: "For fleet validation, is it acceptable to discard everything except the mobile base?"
- Handling: this is where we **ask the agent/human in a structured way**. The tool (conversion tool) is responsible for executing the mechanical conversion; the agent is responsible only for the semantic judgment. **This is not a blanket hand-off** — the mechanical part the tool can own and the semantic-judgment part the agent must own are kept clearly separated.

## Turning Judgment Into an Asset: Record as Simplification Rules and Reapply

- Once a simplification judgment has been made (e.g., "for this use case, keep/discard this element"), it is recorded as a **simplification rule**.
- The next time a similar judgment is needed, the recorded rule is reapplied.
- The more rules accumulate, the less often the agent needs to be consulted.
- The recording format for rules (how to express them, where to store them) is not yet settled — left as an **Open Question** (see [17_Open_Questions_EN.md](./17_Open_Questions_EN.md)).

## Goal-Setting: Not Full Automation, but a Gradual Rise in Automation Rate

- Full automation is not the goal. The goal is:
  - **Automate geometry.**
  - **Raise the automation rate for semantics gradually, through agent assistance plus rule accumulation.**
- This is the same staged approach used for error-recovery automation (rely on human/agent judgment initially, and widen the scope of automation as experience accumulates).

## Related Documents

- [14_Engine_Specific_Adjustment_EN.md](./14_Engine_Specific_Adjustment_EN.md) — The (b) structural-difference conversion this strategy targets
- [11_Design_Principles_EN.md](./11_Design_Principles_EN.md) — The one-directionality principle (detailed → simplified is possible, the reverse is not)
- [17_Open_Questions_EN.md](./17_Open_Questions_EN.md) — Rule recording format; standard for structured questions to the agent
