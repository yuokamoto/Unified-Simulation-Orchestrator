# 4. Behavior Tree Specification

## Purpose
In this framework, the operational logic of robots and assets is described using **Behavior Trees (BT)**,  
allowing the **same logic to be reused across different simulation engines as much as possible**.

By using BT, this framework enables:
- Unified management and reuse of logic (shared across SimPy, Gazebo, Unreal, etc.).
- Flexible choice between running logic as **internal BTs within the simulator** or as **external logic controlled via ROS2/gRPC**.

---

## Formats

**BT XML (BehaviorTree.CPP format)** is the official common format for describing Behavior Trees in this framework.

- Directly usable with Gazebo and C++ BT runtimes via `BehaviorTree.CPP`.  
- Converted into Python classes for execution with the `py_trees` library in SimPy.  
- Editable via GUI tools such as **Groot2**.  
- Other engines can consume BT XML through provided plugins.

> **Note:** SCXML (State Chart XML) is not used as a BT format. While SCXML is a valid state machine standard, it represents a fundamentally different paradigm from Behavior Trees. BT XML was chosen as the single official format to avoid ambiguity in conversion semantics.

---

## Application by Simulation Environment

### SimPy Nodes
- Runs in Python.  
- Executes BTs using the `py_trees` library.  
- Automatically **converts shared BT XML into Python classes**, binding user-defined actions (Python functions) as needed.

### Gazebo Nodes
- Runs in C++.  
- Executes BTs using `BehaviorTree.CPP`.  
- Loads shared BT XML directly and connects each action to C++ functions or ROS2 nodes.

### Other Simulation Engines (Mujoco, Genesis, Unreal, Isaac, etc.)
- Each engine uses an appropriate BT runtime.  
- Shared BT XML can be **converted or read directly** through provided **plugins**.

---

## Integration with the Blackboard
- Each node maintains a **Blackboard**, allowing variables and flags to be shared between BT nodes.  
- The Blackboard is **local to each node and is not synchronized across simulations**.  
  - Exception: **Only essential states (task goals or flags) may be shared via snapshots.**
- Blackboard states can be **optionally included in logs** for debugging and improving reproducibility.

---

## Execution Flow
1. **Load BT File**  
   - Each simulation node loads BTs written in BT XML.
2. **Engine-Specific Conversion**  
   - SimPy: Converts BT XML to `py_trees` Python classes.  
   - Gazebo/Unreal: Loads BT XML directly via `BehaviorTree.CPP`.
3. **Execution and Updates**  
   - BT is updated at each simulation step, determining actions for assets.
4. **External Logic Integration**  
   - Integrates with external algorithms via ROS2/gRPC.

---

## Sample (Simplified BT XML)
```xml
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <Sequence name="MissionSequence">
      <Condition ID="BatteryCheck"/>
      <Action ID="NavigateToTarget"/>
      <Action ID="PickObject"/>
    </Sequence>
  </BehaviorTree>
</root>
