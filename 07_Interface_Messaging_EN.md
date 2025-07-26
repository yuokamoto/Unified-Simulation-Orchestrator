# 7. Interface and Messaging Specification

## Communication Architecture Overview

This framework separates communication paths into **internal (Simulation Master ⇔ Node)** and **external (GUI/control systems ⇔ Simulation Master)**,  
and adopts the following protocol structure:

1. **Simulation Master ⇔ Node (Internal Communication)**
   - Uses **ZeroMQ** as the sole communication mechanism.
   - Provides lightweight, low-latency, and scalable transmission of large volumes of delta snapshots.

2. **External Systems ⇔ Simulation Master (External Communication)**
   - Currently uses ZeroMQ.
   - gRPC and ROS 2 wrappers can be added as future extensions.

---

## Message Types

### 1. Snapshot Synchronization
- Nodes generate **delta snapshots** and send them to the Simulation Master.
- The Simulation Master generates **integrated snapshots** and distributes them to nodes and tools.
- Snapshot structure follows Chapter 3 (schema specification).

**ZeroMQ Message Example (Delta Snapshot)**
```json
{
  "type": "delta_snapshot",
  "timestamp": 123.45,
  "changes": {
    "robot_1": {
      "position": [1.2, 3.4, 0.0],
      "velocity": [0.5, 0.0, 0.0]
    }
  }
}
```

### 2. Simulation Control
- **Step Progression**: Master notifies Nodes to begin the next simulation step.
- **Reset**: Restores state to a specified snapshot.
- **Stop**: Terminates the simulation.

**ZeroMQ Message Example (Step Start)**
```json
{
  "type": "step_start",
  "delta_t": 0.01
}
```

### 3. Event Notification
- Nodes notify the Master of detected events (task completions, BT state transitions, errors).

**ZeroMQ Message Example (Event)**
```json
{
  "type": "event",
  "event_type": "task_completed",
  "asset_id": "robot_1",
  "details": {"task_id": "pickup_001"}
}
```

### 4. Log / Replay Control
- CLI or GUI can instruct the Master to start log playback or recording.
- Commands can be broadcast to all nodes via ZeroMQ.

**ZeroMQ Message Example (Start Replay)**
```json
{
  "type": "start_replay",
  "log_id": "session_2025_07_21",
  "speed": 1.0
}
```

---

## API Structure

1. **ZeroMQ API**
   - Uses lightweight binary or JSON messages for transmitting step control, snapshot synchronization, and event data.

2. **gRPC API**
   - Handles simulation lifecycle management (start, stop, replay) and integration with external monitoring tools.

3. **ROS 2 API**
   - Wraps the gRPC functionality to make it easily accessible from existing ROS 2 nodes.
   - Prefers **zenoh** as the communication middleware.

---

## Implementation Notes

- ZeroMQ messages focus on **delta snapshots only** for efficiency.
- gRPC is limited to **state management and high-level operations** (start/stop/replay).
- ROS 2 integration prioritizes **compatibility with existing simulation engines**, implementing only missing components as needed.
