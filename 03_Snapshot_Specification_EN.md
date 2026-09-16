# 3. Snapshot Specification

## Role
Snapshots represent the state of the simulation world and are used for:
- Initialization: Applying scenario-defined initial states to each simulation engine.
- Synchronization: Merging and synchronizing states across nodes in distributed mode.
- Replay: Replaying simulations or reproducing bugs.
- Logging: Storing and analyzing simulation results.

---

## Types of Snapshots
1. **Full Snapshot**  
   Captures the complete world state (position, velocity, internal states, connections, etc.).
2. **Delta Snapshot**  
   Records only differences from the previous snapshot, reducing bandwidth for large-scale simulations.

---

## Full Snapshot Example (YAML)
```yaml
timestamp: 123.45   # Simulation time (in seconds) represented by this snapshot
world:
  assets:
    <asset_id>:         # Unique identifier for each asset
      type: <string>    # "robot", "human", "object", "conveyor", "elevator", etc.
      model: <string>   # Path to the asset’s 3D/physics model (OpenUSD, URDF, SDF, etc.)
      position: [x, y, z]  # Position in the world coordinate system
      orientation: [qx, qy, qz, qw]  # Orientation as a quaternion
      linear_velocity: [vx, vy, vz]  # Linear velocity vector
      angular_velocity: [wx, wy, wz] # Angular velocity
      joint_positions:  # Non-normative example — exact shape (naming, units, how multi-DOF joints
                        # such as spherical/free joints are keyed) is still open, see Open Question 6.
                        # Omit both joint_positions and joint_velocities for non-articulated assets;
                        # for an articulated asset, a restoration-sufficient full snapshot requires
                        # both together (pose alone cannot resume in-progress motion) — but not
                        # necessarily in this single-float-per-joint shape.
        <joint_name>: <float>   # Illustrative only: a single-DOF (revolute/prismatic) joint's angle
                                 # (rad) or displacement (m). Not valid as-is for multi-DOF joints.
      joint_velocities: # Required together with joint_positions for articulated assets — same
                        # non-normative caveat as above applies
        <joint_name>: <float>   # Illustrative only — see joint_positions above
      status: <string>  # Optional state label (e.g., "idle", "moving", "error")
      connected_to: [<asset_id>, ...]  # List of connected assets (e.g., a robot carrying pallets). Empty list if none.
      properties:       # Internal states or additional attributes
        battery_level: 0.85
        custom_flags:
          carrying_load: true
  reproduction_info:   # Global (not per-asset); required only for consumers that regenerate rendered
                        # images from state instead of storing them — see "Global (Non-Asset) Fields" below
    lighting: <structured lighting description>
    domain_randomization: {<param_name>: <value>, ...}  # Visual/rendering randomization only (e.g.,
                                                          # textures, materials) — dynamics-affecting
                                                          # randomization (friction, mass, ...) is
                                                          # simulation state, not reproduction_info
    seed: <int>          # Illustrative only: an integer alone does not capture RNG algorithm/stream
                          # position after prior draws — see Open Question 8
  meta_data:            # Optional, global, freeform — see "Global (Non-Asset) Fields" below
    <key>: <value>
```

---

## Delta Snapshot Example (JSON)
```json
{
  "type": "state_update",
  "timestamp": 130.00,
  "node_id": "simpy_A",
  "delta_snapshot": {
    "updated_assets": {
      "robot_1": {
        "position": [5.0, 1.0, 0.0],
        "linear_velocity": [0.5, 0.0, 0.0],
        "status": "moving",
        "connected_to": ["pallet_1"]
      }
    },
    "removed_assets": ["human_2"],
    "new_assets": {
      "robot_3": {
        "type": "robot",
        "model": "urdf/robot3.urdf",
        "position": [0.0, 0.0, 0.0],
        "orientation": [0, 0, 0, 1],
        "linear_velocity": [0, 0, 0],
        "angular_velocity": [0, 0, 0],
        "status": "idle",
        "connected_to": []
      }
    },
    "updated_reproduction_info": {
      "seed": 42
    },
    "updated_meta_data": {
      "experiment_tag": "run_017"
    }
  }
}

```

> **Non-normative:** the `updated_reproduction_info` / `updated_meta_data` keys above illustrate intent only. Whether an omitted field or key is retained, replaced, or deleted on the receiving end is not yet defined — see Open Question 8. Do not treat this example as settling that semantics.

---

## Representing Connections
- The `connected_to` field is a **list of asset IDs**, representing *logical* connections without physical constraints.
- This allows expressing states such as:
  - A robot carrying multiple pallets: `connected_to: ["pallet_1", "pallet_2"]`
  - A shelf holding multiple items: `connected_to: ["item_1", "item_2", "item_3"]`
  - A conveyor with packages: `connected_to: ["package_1", "package_2"]`
- An empty list `[]` indicates no connections.
- This enables logical relationship modeling even without running full physics simulation.

---

## Global (Non-Asset) Fields

Not everything a snapshot carries is per-asset state. Two additional fields live directly under `world`, alongside `assets`:

- **`reproduction_info`** — Structured, typed data needed specifically to regenerate a rendered image from state without storing the image itself: **scene-wide, rendering-only** settings — lighting and *visual* domain-randomization (textures, materials, camera/sensor noise, and the like) — plus the random seed used. This deliberately excludes any domain-randomization that affects dynamics (friction, mass, actuator gains, and the like): those change how the simulation continues, so they are simulation state, not reproduction info, and must not be treated as optional. Precisely because dynamics-affecting randomization is excluded, this field is *not* required to continue the simulation. It is episode-scoped and normally unchanged for the duration of a run, so it is typically carried once in the full snapshot rather than repeated in every delta.
  - **Cameras are not part of `reproduction_info`.** A camera used for observation is itself an asset, not global state: a sensor rigidly mounted on a robot (e.g., a wrist camera) has its world pose already derivable from the parent asset's tracked state — its root pose and, for an articulated asset, its already-tracked `joint_positions` — combined with the fixed mounting offset and kinematic chain defined in the asset's model, so it needs no separate snapshot field at all (a single fixed offset from the root pose is only sufficient when the camera is mounted directly on a non-articulated body); a free-standing observation camera (e.g., a fixed overview camera) is tracked like any other asset (`type: "camera"`, with `position`/`orientation` as usual), with its intrinsics held as a static property of that asset's model rather than as per-step data — though the exact contract for where/how that calibration (focal length, resolution, field of view) is expressed is not yet standardized (`model` here is only a path; no calibration schema or lookup convention is defined), so this exclusion is complete only once that static-camera contract exists — see Open Question 11. See [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md).
- **`meta_data`** — A freeform, user-defined bag of global (simulation-wide) data that is not required for any restoration purpose — the global-scope counterpart to the existing per-asset `properties` field. Use it for experiment tags, debug flags, or other custom data; its shape is intentionally unconstrained, so nothing in the framework or in an external consumer should depend on it being present or on any particular key existing.

Both fields are optional. See [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md) for the design discussion behind this split, and [17_Open_Questions_EN.md](./17_Open_Questions_EN.md) for what remains unresolved (exact sub-field shapes).

---

## Operational Policy
- Snapshots primarily describe the world state (per-asset position, velocity, and other dynamic state; updates are handled within each simulator's logic). The global `reproduction_info` and `meta_data` fields (see "Global (Non-Asset) Fields" above) are the deliberate exception: they carry rendering/reproduction configuration and user metadata, not simulated world state.
- Use cases:
  1. **Initialization** – Apply at simulation start.
  2. **Synchronization** – Share state between nodes in distributed mode.
  3. **Logging** – Store historical states for replay and analysis.

---

## Related: Design Discussion Record

The positioning of the snapshot as the source of truth for "state," and its role at the boundary with the training pipeline (the state/data supply API), are recorded in [13_Reuse_Layers_and_Format_Selection_EN.md](./13_Reuse_Layers_and_Format_Selection_EN.md) and [12_Layering_and_ML_Boundary_EN.md](./12_Layering_and_ML_Boundary_EN.md).

The `joint_positions` / `joint_velocities` fields above (and their exact shape) are a still-open extension surfaced by an external consumer of this format; see [18_Snapshot_Alignment_with_Learning_Data_EN.md](./18_Snapshot_Alignment_with_Learning_Data_EN.md) and [17_Open_Questions_EN.md](./17_Open_Questions_EN.md).

The `reproduction_info` and `meta_data` global fields above were added to settle where non-per-asset data (rendering/reproduction settings, user-arbitrary tags) belongs; see [19_Snapshot_MetaData_and_Reproduction_Info_EN.md](./19_Snapshot_MetaData_and_Reproduction_Info_EN.md).
