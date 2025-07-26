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
      status: <string>  # Optional state label (e.g., "idle", "moving", "error")
      connected_to: <asset_id or null>  # Other asset it is connected to (e.g., a robot carrying a pallet)
      properties:       # Internal states or additional attributes
        battery_level: 0.85
        custom_flags:
          carrying_load: true
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
        "connected_to": "pallet_1"
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
        "connected_to": null
      }
    }
  }
}

```

---

## Representing Connections
- The `connected_to` field represents *logical* connections without physical constraints.
- This allows expressing states like a robot carrying a pallet even without running full physics simulation.

---

## Operational Policy
- Snapshots **only describe the world state**; position and velocity updates are handled within each simulator’s logic.
- Use cases:
  1. **Initialization** – Apply at simulation start.
  2. **Synchronization** – Share state between nodes in distributed mode.
  3. **Logging** – Store historical states for replay and analysis.
