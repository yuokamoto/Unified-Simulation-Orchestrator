# 2. Unified Architecture

## Layered Structure

This framework is designed to handle multiple types of simulation engines in a unified manner,  
enabling the reuse of assets and logic through the following layers:

1. **Snapshot Management Layer**  
   - Manages the world state as **Full Snapshots** (the complete state at time `t`) and **Delta Snapshots** (differences from the previous step).  
   - Serves as the foundation for initialization, state synchronization, logging, and replay.  
   - In distributed mode, uses key-value stores such as Redis to facilitate node synchronization and recovery.

2. **Simulation Layer**  
   - The layer where the actual simulation takes place.  
   - Allows selecting engines such as **SimPy, Gazebo, Mujoco, Genesis, Isaac Sim, Unreal Engine** depending on the use case.  
   - Can switch between logic-only simulation (physics disabled) and high-precision physics or high-quality 3D visualization.  
   - Handles objects such as conveyors and elevators with optional physics.

3. **Logic Control Layer**  
   - **Internal Logic Control**: Controls assets (robots, humans, etc.) within a node using Behavior Trees (BT).  
   - **External Logic Control**: Enables integration with external applications and real robot algorithms via ROS2 or gRPC.

4. **Interface Layer**  
   - Abstracts communication between the Simulation Layer and external control/state management.  
   - **Uses gRPC as the core communication protocol**, with a ROS2 wrapper available as needed.  
   - Provides unified APIs for snapshot synchronization, event notifications, and control commands.

5. **Tools Layer**  
   - **Snapshot Editing Tools**: Edit world states via GUI or scripts.  
   - **Scenario Editor**: Create and edit YAML/JSON scenarios, with the ability to reference and preview snapshots.  
   - **Behavior Tree Editor**: Edit and export shared BT definitions.  
   - **Format Conversion Tools**: Convert ROSBag and other data formats into snapshots.

---

## Data Flow and File Relationships

1. **Scenario Files (YAML/JSON)**  
   - Describe the "intent and flow" of the simulation.  
   - Include mission goals, task schedules, and event triggers.  
   - **Contain references to initial state snapshots** to explicitly define the starting world state.

2. **Snapshot Files (YAML/JSON)**  
   - Capture the "actual world state" at a specific time.  
   - Contain positions, orientations, velocities, internal states, and connections (e.g., a robot carrying a pallet).  
   - Used for initialization, synchronization between distributed nodes, bug reproduction, logging, and replay.

3. **Behavior Trees (BT)**  
   - Defined in SCXML or BT XML as a common format, automatically converted by each node for execution.  
   - For example: SimPy uses `py_trees`, while Gazebo or other engines may use `BehaviorTree.CPP`.

4. **During Simulation Execution**  
   - Scenario → Load referenced initial snapshot → Start simulation.  
   - Each node generates delta snapshots at each step and sends them to the Simulation Master.  
   - The Simulation Master manages the integrated state and, when necessary, stores logs for replay and analysis.

---

## Operational Modes

1. **Single Mode**  
   - Runs the Master and Node within a single process, eliminating inter-process communication.  
   - Suited for development and small-scale testing.

2. **Distributed Mode**  
   - Runs multiple nodes (SimPy, Gazebo, other engines) across a cluster of servers.  
   - The Simulation Master synchronizes states using delta snapshots and manages the integrated world state.

3. **Multi-Engine Switching**  
   - Scenarios, snapshots, and BT definitions remain shared while only the simulation engine is switched.  
   - Example workflow:  
     - Large-scale logic validation in SimPy → Physics validation in Gazebo → Demo rendering in Unreal Engine.
