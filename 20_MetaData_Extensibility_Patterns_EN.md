# 20. Extensibility Patterns for `meta_data` / `properties`: Prior Art and Candidate Approaches

**Status:** Draft — Exploration, Not Decided
**Date:** 2026-09-16

> This document does not settle a design decision. It records prior art surveyed during a design discussion and the candidate approaches it surfaced, for whoever picks up Open Question 9 in [17_Open_Questions_EN.md](./17_Open_Questions_EN.md). [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md) already settled that `reproduction_info` (core, typed, required by feature) and `meta_data`/`properties` (freeform, no guarantees) are kept structurally distinct; this document explores a further question raised by that decision.

## The Question

[19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md) established that folding `reproduction_info` into `meta_data` (by giving a specific key inside `meta_data` its own required-field schema) would not remove the required/freeform distinction — it would only relocate it one level deeper. That framing raises a genuinely separate, forward-looking question: should USO ever let a *specific external consumer* (not USO core) register its own named, typed schema for data it attaches inside `meta_data` (or the per-asset `properties`), so that consumer gets the same reliability guarantees (validation, required fields) that `reproduction_info` gets today — without USO core having to know about that consumer's schema in advance?

This is not needed for any currently known requirement (`reproduction_info`, joint state). It is recorded here only because a real internal precedent and multiple established open-source patterns for exactly this problem surfaced during the discussion, and are worth keeping close to the design so they aren't rediscovered from scratch later.

## Reference: Prior Art

Variations on the same underlying idea — **a type/kind tag identifies which schema a payload follows, and the schema for a given type is owned and versioned by whoever defined that type** — recur across widely used systems, though the systems differ in the details (whether an unrecognized type is silently ignored, pruned, or rejected; and whether every payload even carries an explicit type tag):

- **Kubernetes** — `labels`/`annotations` (freeform, namespaced `<domain>/<name>` keys, no validation) vs. **CustomResourceDefinitions** (a `kind` gets its own registered OpenAPI v3 schema, with `required` fields and validation, optionally preserving unknown fields).
- **Protocol Buffers** — `google.protobuf.Any` (`type_url` + serialized payload; only a consumer that knows the type unpacks it) and the `Struct`/`Value` well-known types (an officially blessed "freeform JSON" escape hatch, contrasted with strongly-typed messages).
- **CloudEvents** (CNCF) — an event carries a `type` (identifying the kind of event) and a `data` payload; an optional, separate `dataschema` attribute points to the schema `data` follows, owned and versioned by whoever defined it; consumers that don't recognize the `type` (or lack the schema) can still route/store the event opaquely.
- **OCI** (container/artifact spec) — `mediaType` / `artifactType` fields declare how to interpret an otherwise opaque payload.
- **JSON Schema `oneOf` + OpenAPI 3's `discriminator`** — `oneOf` (a JSON Schema keyword) requires a value to match exactly one of several schemas, and is what actually performs the validation; `discriminator` (an OpenAPI-specific extension, not part of JSON Schema itself) does not add a field — it names an *existing* property in the payload (via `propertyName`) and optionally maps that property's values to specific schemas, letting tooling pick the right branch directly instead of testing each `oneOf` alternative.
- **glTF** (Khronos) — a core spec plus a registered `extensions` namespace; `extensionsUsed` / `extensionsRequired` arrays let a file explicitly declare which extensions a loader *must* understand to interpret it correctly, versus which are safe to ignore.
- **OpenUSD** (Pixar) — directly relevant since USO already depends on it for assets: plugin-registered **IsA / API schemas** define typed attribute sets with fallback/default values (USD's own schema/defaulting semantics — not JSON-Schema-style enforced-required validation), while **`customData`/`assetInfo`** remain freeform dictionaries for anything else. This is the same two-tier split USO has already made for `reproduction_info` vs. `meta_data`, native to a format USO already uses.
- **OpenTelemetry** semantic conventions — namespaced, well-known attribute keys coexist with arbitrary custom attributes on the same object.
- **Confluent / Apicurio Schema Registry** — a central registry maps a subject (≈ type name) to a versioned schema and enforces compatibility rules across versions, informing how USO might version `reproduction_info` or the per-joint fields ([17_Open_Questions_EN.md](./17_Open_Questions_EN.md) items 6 and 8) without breaking existing consumers.

## Candidate Approaches for USO (Unordered, Not Decided)

- **Namespace convention only** — require keys inside `meta_data`/`properties` to follow `<domain>/<name>` (as Kubernetes annotations do). Zero infrastructure; solves collisions between consumers but not validation/required-ness.
- **Typed-value wrapper** — entries shaped like `{"type": "<name>", "value": {...}}` (the `Any` pattern), so a consumer that recognizes `type` can validate `value` against a schema it owns.
- **Snapshot-level `schema_version`** — a version marker on the snapshot format itself, enabling `reproduction_info`'s or the joint-state fields' shape to evolve with a defined compatibility/conversion story, independent of the `meta_data` question.
- **External registered schema** (CRD-like) — a consumer publishes a schema (e.g., JSON Schema) for its own named extension in a known location; tooling validates any `meta_data`/`properties` entries matching that name against it, and leaves everything else untouched.

Choosing between (or combining) these, and deciding whether this is worth building before a real external consumer asks for it, is left open.

## Related Documents

- [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md) — The decision this document explores a follow-on question to
- [03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md) — Where `meta_data`/`properties` are defined today (freeform, no schema)
- [17_Open_Questions_EN.md](./17_Open_Questions_EN.md) — Open Question 9, recorded from this exploration
