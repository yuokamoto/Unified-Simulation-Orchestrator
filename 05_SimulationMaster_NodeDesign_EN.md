# 5. Simulation Master and Node Design

## Purpose and Role of the Simulation Master

The **Simulation Master** is the core component introduced primarily to enable **Distributed Simulation**.  
It orchestrates multiple simulation nodes (SimPy, Gazebo, Mujoco, Genesis, Unreal, Isaac, etc.) running across multiple servers,  
synchronizing time and state while managing the overall simulation coherently.

In **Single Mode** (running on a single node/machine), many of these functions are unnecessary,  
and the Master operates in a simplified form to minimize overhead.

---

## Roles of the Simulation Master (Distributed Mode)

1. **Time Synchronization and Step Control**  
   - Synchronizes simulation time (Δt steps) across all nodes.  
   - Waits for all nodes to complete each step and performs barrier synchronization before advancing.

2. **Snapshot Integration and Management**  
   - Integrates delta snapshots received from nodes into a global snapshot.  
   - Caches full snapshots in Redis to support node reconnection and replay.

3. **Node Registration and Health Monitoring**  
   - Tracks node connections/disconnections and monitors health via heartbeats.  
   - If a node crashes and restarts, it can quickly resynchronize using cached snapshots.

4. **Behavior Tree (BT) Distribution and Initialization**  
   - Distributes shared BT XML to all nodes and initializes local BT runtimes.

5. **External API Provision**  
   - Provides a **gRPC-based external API** for GUI, CLI, and third-party tool integration.  
   - A **ROS2 wrapper** is available for seamless integration with existing ROS2 node ecosystems.  
   - Internal Master ⇔ Node communication uses **ZeroMQ** exclusively.

6. **Logging and Replay Management**  
   - Stores global snapshots and event logs for offline replay and analysis.

---

## Simplified Behavior in Single Mode

In Single Mode, the Simulation Master and Node are **integrated into a single process**,  
and operate with the following simplifications:

- **Time synchronization and barrier control are omitted**, with the Master calling `step(delta_t)` directly.  
- Snapshot integration and Redis caching are unnecessary (handled entirely within the node).  
- Node registration and health monitoring are disabled (not needed within a single process).  
- BT XML is loaded locally without distribution.  
- Logs and replays are simply stored on local disk.

This enables **high performance for small-scale scenarios and development/debugging use cases**.

---

## Common Node Interface

All simulation nodes must implement the following interface:

```python
class SimulationNode:
    def initialize(self, full_snapshot, bt_xml):
        """Load the initial snapshot and construct the BT tree"""
        pass

    def wait_for_sync(self):
        """Distributed mode: Wait for the Master’s signal to start a step.
        Single mode: Not required."""
        pass

    def step(self, delta_t):
        """Tick the BT and advance the simulation engine by one step"""
        pass

    def collect_asset_states(self):
        """Collect the current asset states and return as a delta snapshot"""
        pass

    def send_state_update(self, delta_snapshot):
        """Send delta snapshots to the Master (local handling in single mode)"""
        pass

    def send_event(self, event):
        """Send events to the Master (local handling in single mode)"""
        pass

    def shutdown(self):
        """Save the final snapshot and release resources"""
        pass
```

---

## Internal Node Loop Design (Examples)

### SimPy Node
- Handles lightweight, logic-focused simulations.  
- Processes BT using `py_trees` and advances time using SimPy’s `Environment`.

### Gazebo Node
- Handles high-fidelity physics simulations.  
- Processes BT using `BehaviorTree.CPP` and steps Gazebo’s physics engine in sub-steps.

### Other Engines (Mujoco, Genesis, Unreal, Isaac, etc.)
- Any engine can integrate with the Master as long as it implements the `SimulationNode` interface.

---

## Communication and Synchronization Cycle in Distributed Mode

1. The Master sends a "step start" signal to all nodes.  
2. Each node executes `step(delta_t)`, performing BT ticks and simulation updates.  
3. Each node generates a delta snapshot and sends it to the Master.  
4. The Master composes an integrated snapshot, caching and logging as needed.  
5. Barrier synchronization ensures all nodes align before proceeding to the next step.

Through this cycle, **consistent simulation state and time management** is achieved even in distributed environments.

---

## Asset Ownership Rules (Distributed Mode)

In distributed mode, **each asset is owned by exactly one node** at any given time.  
This ownership model prevents conflicting updates and ensures deterministic state management.

1. **Single-Owner Principle**  
   - Each asset (robot, human, object, etc.) is assigned to one and only one simulation node.  
   - Only the owning node may update the asset's state (position, velocity, joint_positions, joint_velocities, status, connections, properties) in delta snapshots — this list follows the asset schema in [03_Snapshot_Specification_EN.md](./03_Snapshot_Specification_EN.md) and grows with it; it is not a fixed enumeration.  
   - Other nodes receive the asset's state as read-only via the integrated snapshot from the Master.

2. **Ownership Assignment**  
   - Ownership is determined at initialization based on the scenario and initial snapshot.  
   - The Simulation Master maintains the ownership registry and distributes it to all nodes.

3. **Ownership Transfer**  
   - When an asset needs to be handed off between nodes (e.g., a pallet moving from a SimPy logistics zone to a Gazebo physics zone), an explicit **ownership transfer request** is sent to the Master.  
   - The Master coordinates the transfer, ensuring no two nodes update the same asset simultaneously.  
   - During transfer, the asset's full state is synchronized to the new owner via a snapshot.

4. **Conflict Detection**  
   - If the Master receives delta snapshots from multiple nodes updating the same asset, it treats this as an error and logs a conflict warning.  
   - The owning node's update takes precedence; non-owner updates are discarded.
