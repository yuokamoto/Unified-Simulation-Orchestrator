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
   - Provides a gRPC-based core API and, when needed, a ROS2 wrapper for external control, GUI, and debugging tools.

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
