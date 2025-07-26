
# Unified Simulation Orchestrator (USO) – Implementation Task List

This document outlines all implementation tasks for building the **Unified Simulation Orchestrator (USO)** based on the finalized specifications and architecture.

---

## 1. Foundation Setup

1. **Repository Initialization and Structure**
   - Set up a mono-repo for Python (SimPy nodes, snapshot manager), C++ (Gazebo nodes, BT runtime), and JavaScript (GUI).
   - Follow directory structure outlined in Chapter 9.

2. **Build System Configuration**
   - Python: `poetry` or `pipenv`.
   - C++: `CMake` + `vcpkg`.
   - JavaScript: `npm` + `vite`.
   - Automate gRPC code generation (Python/C++/JS).
   - Ensure Bazel compatibility for future expansion.

3. **Shared Data Model Definitions**
   - Implement YAML/JSON schemas for snapshots (Chapter 3) and scenarios (Chapter 2).
   - Establish asset references for OpenUSD and URDF/SDF.

---

## 2. Communication and Synchronization

4. **ZeroMQ Messaging Library**
   - Implement internal Master ⇔ Node communication (Chapter 7).
   - Support delta snapshots, step control, and event notification.

5. **Redis Snapshot Cache**
   - Store and retrieve full snapshots for recovery and replay.

6. **gRPC API with ROS 2 Wrapper**
   - Provide simulation control API for external tools.
   - Wrap with `simulation_interfaces` for ROS 2 compatibility.

---

## 3. Simulation Master Implementation

7. **Core Simulation Master**
   - Manage node registration, health checks, and heartbeat monitoring.
   - Control step synchronization (fixed Δt, barrier sync).
   - Aggregate delta snapshots and save to Redis.
   - Handle global logging (snapshots and events).

8. **Single Mode Execution**
   - Run Master and Node in one process for small-scale testing.
   - Bypass messaging for performance.

---

## 4. Simulation Node Development

9. **SimPy Node**
   - Implement lightweight logic simulation loop using SimPy.
   - Integrate `py_trees` for Behavior Tree execution.
   - Generate delta snapshots and send via ZeroMQ.

10. **Gazebo Node**
    - Use Gazebo with BehaviorTree.CPP for physics simulation.
    - Load URDF/SDF and YAML snapshots.
    - Generate delta snapshots and send via ZeroMQ.

11. **Future Engine Plugins**
    - Implement `SimulationNode` interface for Mujoco, Genesis, Unreal, and Isaac Sim.

---

## 5. Logic and Scenario Management

12. **Behavior Tree Conversion Tools**
    - Convert SCXML → `py_trees` (Python).
    - Convert SCXML → BehaviorTree.CPP XML (C++).
    - Integrate Groot2 for GUI editing.

13. **Scenario Parser and Snapshot Initialization**
    - Load initial snapshots referenced in scenarios.
    - Provide event scheduling for simulation nodes.

---

## 6. GUI and Tooling

14. **Web GUI (Electron + React + Three.js)**
    - 3D visualizer for OpenUSD assets.
    - Integrate real-time delta snapshots for visualization.
    - Scenario editor for YAML/JSON (with snapshot previews).
    - Behavior Tree editor (drag-and-drop).
    - Log and replay viewer.

15. **Support Tools**
    - ROSBag ↔ YAML snapshot conversion scripts.
    - GUI for snapshot editing.
    - Log analysis and visualization tools.

---

## 7. Testing and Integration

16. **Unit Tests**
    - Validate snapshot serialization/deserialization.
    - Test ZeroMQ message handling.
    - Verify BT conversion workflows.

17. **Integration Tests**
    - Single-mode simulation: SimPy + Master.
    - Distributed mode: multi-node SimPy cluster.
    - Reuse scenarios with Gazebo integration.

18. **CI/CD Pipeline**
    - GitHub Actions for build and test automation.
    - Docker Compose for distributed simulation environments.

---

## 8. Future Enhancements

19. **Omniverse Integration**
    - Leverage USD Live Sessions for cross-tool collaboration.

20. **Dynamic Load Balancing**
    - Distribute computational load dynamically across nodes.

---

### Usage
- Each section can be delegated to AI-assisted coding incrementally.
- Reference **existing specs for APIs (ZeroMQ, YAML schemas, gRPC)** to ensure consistency.
