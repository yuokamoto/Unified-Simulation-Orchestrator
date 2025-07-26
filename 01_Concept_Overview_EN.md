# 1. Project Concept and Overview

## Purpose
**Unified Simulation Orchestrator (USO)** is a  
**unified simulation framework that allows switching and integrating multiple types of simulation engines (SimPy, Gazebo, Mujoco, Genesis, Isaac Sim, Unreal Engine, etc.),  
while maximizing the reuse of assets (models), logic, and scenarios**.

Primary objectives:
- Efficiently validate **large-scale and scalable scenarios** (warehouses, factories, robotics) using lightweight logic-driven simulations.
- Integrate **high-precision physics simulations and 3D visualizations** when needed.
- Use **common formats (OpenUSD, URDF/SDF, Behavior Tree XML, YAML snapshots)** to share assets, logic, and scenarios across multiple engines.
- Support **both single-node and distributed modes**, enabling seamless scaling from small tests to large-scale experiments with thousands of agents.

## Core Concepts
1. **Multi-Engine Support and Switching**  
   - Use **SimPy** for lightweight, event-driven, logic-centric validation.  
   - Use **Gazebo, Mujoco, Genesis** for high-fidelity physics and ROS2 integration.  
   - Use **Isaac Sim, Unreal Engine** for high-quality 3D rendering, marketing demos, and ML dataset generation.  
   - **Scenarios and logic are centrally managed, allowing engines to be switched without rewriting core content.**

2. **Reusability of Assets and Logic**  
   - **OpenUSD** for unifying 3D models, usable across all rendering pipelines.
   - **URDF/SDF** for defining robot structures, reusable in Gazebo and other engines.
   - **Common Behavior Tree (SCXML/BT XML)** to execute the same logic in SimPy, Gazebo, Unreal, and others.

3. **Snapshot-Centric State Management**  
   - Represent world state with **full snapshots** and **delta snapshots**,  
     enabling unified initialization, synchronization, bug reproduction, and replay.

4. **Flexible Operational Modes**  
   - **Single Mode**: Optimized for fast debugging or small-scale tests on a single machine.  
   - **Distributed Mode**: Enables scalable, fault-tolerant, multi-server simulations for large-scale workloads.
