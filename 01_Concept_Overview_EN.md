# 1. Project Concept and Overview

## Purpose
The **Unified Simulation Orchestrator (USO)** aims to provide  
a **unified simulation framework that can switch between and integrate multiple types of simulation engines (SimPy, Gazebo, Mujoco, Genesis, Isaac Sim, Unreal Engine, etc.) while maximizing the reuse of assets (models), logic, and scenarios.**

The primary objectives are as follows:
- **Efficiently validate large-scale and scalable scenarios** (warehouses, factories, robotics) using lightweight logic-driven simulations.
- **Integrate high-fidelity physics simulations and 3D visualization** as needed.
- **Leverage common formats (OpenUSD, URDF/SDF, Behavior Tree XML, YAML snapshots)** to share assets, logic, and scenarios across multiple engines.
- **Support both single-mode and distributed-mode**, enabling seamless scaling from small-scale tests to experiments involving thousands of agents.
- **Preserve the user experience of existing simulators**. When operating in single-mode, the framework minimizes changes so that users can continue using their familiar workflow. For example, with Gazebo, the simulation framework can still be launched via `roslaunch`, with only a few additional nodes or processes, without significant workflow disruption.

## Background
In robotics development, simulations are used for a variety of purposes, each requiring different capabilities from the simulator. Representative examples are as follows:

| User                        | Requirements                                                                 | Recommended Simulators                                                                 |
| --------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **Navigation & Robot Arm Engineers** | High-accuracy physics and sensor models (LiDAR, IMU, etc.) and collision detection | **Isaac Sim**, **Gazebo (Ignition)**, **Webots**, **Mujoco**                           |
| **Multi-Robot Development Engineers** | Lightweight, scalable multi-agent support, distributed execution, skipping unnecessary physics | **SimPy**, **Stage**, **Flatland**, **ROS 2 + custom simulator**                       |
| **ML Teams**                | High-quality 3D rendering, camera image generation, synthetic data support | **Unreal Engine**, **Unity**, **NVIDIA Omniverse**                                     |
| **Sales & Consulting**      | Customer-specific deployment simulations, non-engineer-friendly UX, integration with external devices and workflows, scalable multi-agent support | **FlexSim**                                                                            |
| **Marketing**               | High-quality 3D rendering and visualization of robots and cargo for product presentations | **Unreal Engine**, **Unity**, **NVIDIA Omniverse**                                     |

The optimal simulator depends on the user's goals (physics fidelity, scalability, visualization, etc.), and there is no single simulator that can satisfy all these requirements at a high level. As a result, teams often either:
- Switch between different tools depending on the user or development stage, or
- Force the use of simulators that are ill-suited for the task.

Additionally, when using multiple simulators, teams often need to recreate the same logic and assets for each simulator.

Even if a new simulator is developed, replacing existing ones faces challenges:
- Users must learn new tools and workflows.
- Existing simulation tools and environments must be rebuilt.

These changes often meet resistance from team members, making migration difficult.

Therefore, instead of aiming for a single simulator to handle everything, this framework proposes a **simulation orchestration layer that can switch between multiple simulators while maximizing the reuse of logic and assets.** Furthermore, the framework is designed to **preserve the familiar experience of existing simulators** so that users can adopt it with minimal burden, while continuing to leverage existing tools.

## Core Concepts
1. **Multi-Engine Support and Switchability**  
   - Use **SimPy** for lightweight, logic-driven, event-based validation.  
   - Use **Gazebo, Mujoco, Genesis** for high-fidelity physics and ROS 2 integration.  
   - Use **Isaac Sim, Unreal Engine** for high-quality 3D rendering, marketing demos, and ML dataset generation.  
   - **Scenarios and logic are centrally managed, allowing flexible switching of simulation engines as needed.**

2. **Reusability of Assets and Logic**  
   - Standardize 3D models with **OpenUSD**, making them usable across all rendering tools.
   - Describe robot structures in **URDF/SDF**, allowing shared use across Gazebo and other engines.
   - Execute shared logic via **common Behavior Trees (SCXML/BT XML)** across SimPy, Gazebo, Unreal, and others.

3. **Snapshot-Centric State Management**  
   - Represent the world state using **full snapshots** and **delta snapshots**,  
     unifying initialization, synchronization, bug reproduction, and replay workflows.

4. **Flexible Operation Modes**  
   - **Single Mode**: For fast debugging and small-scale testing on a single machine.  
   - **Distributed Mode**: For large-scale simulations across multiple servers, ensuring fault tolerance and scalability.
