# 10. Engine Selection Criteria and Rationale for Distributed Simulation

## Reasons for Adopting Multiple Simulation Engines

1. **Optimization by Use Case**
   - **SimPy**:
     - Python-based, event-driven simulation framework.
     - Lightweight and scalable for large-scale scenarios.
     - Ideal for fast validation of scheduling and process logic without physics computation.
   - **Gazebo**:
     - Easy integration with ROS2, native support for URDF/SDF.
     - Capable of realistic physical simulations including sensors and dynamics.
     - Best suited for motion planning and dynamic interaction (collisions, force dynamics).
   - **Mujoco / Genesis**:
     - High-performance physics engines specialized for robotics research.
     - Suitable for fine-grained dynamics and control algorithm studies.
   - **Isaac Sim / Unreal Engine**:
     - Enables high-quality 3D visualization, marketing demos, and ML dataset generation.

2. **Shared Assets and Logic**
   - **Open-USD**:
     - Expresses complex 3D assets, materials, and animations.
     - Supported by major rendering pipelines like Omniverse and Unreal.
     - Facilitates shared visualization assets across multiple engines.
   - **URDF / SDF**:
     - Standard formats describing the physical properties of robots and environments.
     - Highly compatible with Gazebo and ROS2, convertible for other engines.
   - **YAML (Snapshots)**:
     - Human-readable, easily handled across Python and C++.
     - Captures time-series world states (position, velocity, status, connections).
   - **Behavior Tree (BT) Common XML**:
     - Conforms to `BehaviorTree.CPP` and convertible to `py_trees`.
     - Editable via GUI tools like Groot2 and shareable between C++ and Python.

3. **Separation of Scenario and Snapshot**
   - **Scenario**:
     - Describes tasks, events, and mission intent.
     - References an initial snapshot to define the world’s starting state.
   - **Snapshot**:
     - Captures the world state at a specific time for initialization, synchronization, and replay.
     - Engine-independent format.

---

## Rationale for Communication and Middleware Choices

1. **gRPC (Core Communication)**
   - Provides fast, type-safe, bidirectional communication.
   - Supports multiple languages (Python, C++, JavaScript), unifying diverse stacks within the framework.
   - Enables streaming and bidirectional updates (simulation states, event notifications).

2. **ROS 2 (as a Wrapper)**
   - The standard communication backbone in robotics.
   - Simplifies integration with real robots and existing ROS2 node networks.
   - Used as a wrapper rather than the core, **reducing complexity inside the simulation while preserving external connectivity**.

3. **ZeroMQ (Internal State Synchronization)**
   - Optimized for fast, lightweight messaging in distributed simulations.
   - Lower latency than gRPC, suited for frequent state transfers and delta snapshot delivery between nodes.
   - Effective for high-throughput use cases (synchronization of deltas).

---

## Reasons for Distributed Simulation

1. **Scalability**
   - Factory and warehouse scenarios with hundreds or thousands of agents  
     cannot be handled by a single process, requiring multi-node distribution.

2. **Flexible Resource Allocation**
   - Physics-heavy or rendering nodes can run on GPU servers,  
     while lightweight logic nodes can run on CPU servers.

3. **Fault Tolerance**
   - Nodes can recover using cached snapshots in Redis when failures occur.
   - Ensures stability for long-running simulations.

4. **Efficient Team Development**
   - Logic validation and physics testing teams can work concurrently.
   - Distributed setups allow simultaneous execution of large-scale scenarios and small-scale unit tests.

---

## Importance of Single Mode
- For small-scale tests and debugging, **Simulation Master can be simplified, with nodes running in a single process**.  
- Eliminates communication and synchronization overhead, enabling faster simulation.  
- Maintains the same architecture, allowing seamless migration from single mode to distributed mode.

---

## Design Goals Achieved by These Choices

1. **Scalability**  
   - Lightweight distributed logic via SimPy and detailed physics via Gazebo/Mujoco,  
     flexibly addressing scale and precision requirements.

2. **Reusability**  
   - Shared Open-USD, URDF/SDF, YAML snapshots, and common BT enable cross-engine asset and logic reuse.

3. **Communication Efficiency**  
   - External connectivity via gRPC and high-speed internal messaging via ZeroMQ provide a flexible, high-performance distributed architecture.

4. **Usability**  
   - GUI-based BT editing via Groot2, human-readable YAML, and open-standard formats  
     create an environment accessible to non-engineers.
