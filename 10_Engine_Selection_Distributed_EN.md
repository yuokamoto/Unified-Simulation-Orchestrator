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

## Simulator Landscape Trends (2025–2026) and Why USO Stays Framework-Agnostic

> Recorded from a later design discussion (2026-08-31); see also [11_Design_Principles_EN.md](./11_Design_Principles_EN.md).

The physics/training-simulator ecosystem keeps moving, which is itself an argument for treating the simulator layer as swappable rather than betting on one:

- **Rendering-free, state-based policy training** (e.g., MuJoCo-based MJX/Playground) can run at very high step-rates across platforms, and much current RL/VLA evaluation tooling assumes this style.
- **Large-scale parallel locomotion/manipulation** (GPU-parallel) is a strength of Isaac Lab.
- **Photoreal perception training and synthetic data** (rendering required) remains a strength of Isaac Sim / Replicator.
- **Convergence in progress**: Newton (an NVIDIA/DeepMind/Disney collaboration targeting physics accuracy together with GPU parallelism, on a MuJoCo-Warp backend) and mjlab (pairing the Isaac Lab API with MuJoCo's lightness) suggest the boundaries between these tools may blur over the next year or two.
- **Implication for USO**: lean on USD for assets so they aren't wasted across a MuJoCo ↔ Newton-style transition, but avoid over-committing the framework layer to any single one of these. USO's multi-engine switching capability is itself the hedge against this churn — a concrete instance of why [Principle 1 in 11_Design_Principles_EN.md](./11_Design_Principles_EN.md) treats "which training framework is in fashion" as a reason for change that must stay decoupled from assets and task logic.

---

## Rationale for Communication and Middleware Choices

1. **gRPC (External Communication)**
   - Provides fast, type-safe, bidirectional communication.
   - Supports multiple languages (Python, C++, JavaScript), unifying diverse stacks within the framework.
   - Enables streaming and bidirectional updates (simulation states, event notifications).
   - Serves as the core protocol for GUI, CLI, and third-party tool integration with the Simulation Master.

2. **ROS 2 (as a Wrapper)**
   - The standard communication backbone in robotics.
   - Simplifies integration with real robots and existing ROS2 node networks.
   - Used as a wrapper over gRPC rather than the core, **reducing complexity inside the simulation while preserving external connectivity**.

3. **ZeroMQ (Internal State Synchronization)**
   - The sole communication mechanism for internal Master ⇔ Node messaging.
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
   - External connectivity via gRPC and high-speed internal messaging via ZeroMQ provide a clear separation of concerns and a high-performance distributed architecture.

4. **Usability**  
   - GUI-based BT editing via Groot2, human-readable YAML, and open-standard formats  
     create an environment accessible to non-engineers.
