# 8. GUI Design (Web + Electron-Based)

## Purpose
The GUI functions as an **integrated tool for visualization, scenario editing, replay, and debugging** of the simulation.  
It is primarily **web-based (React + Three.js/Babylon.js)** and accessible via browsers.

---

## Communication Architecture

### Role of Communication Protocols

- **Frontend (GUI ⇔ Web Server / Simulation Master)**  
  - Utilizes **WebSocket or gRPC-Web** for bidirectional communication.  
  - The Web Server bridges with the Simulation Master's gRPC API and relays snapshots/events to the frontend via WebSocket.

---

## System Composition

### Candidate Technology Stack
- **Frontend**: React (UI) + Three.js (3D visualization)
- **Backend**: Web server built with Node.js or Python  
  - Operates as a WebSocket/gRPC-Web server

---

## Functional Modules

### 1. 3D Visualizer
- Loads OpenUSD assets to render robots, humans, and equipment in 3D.
- Receives delta snapshots via WebSocket for real-time scene updates.

> **TODO:** Three.js has no native OpenUSD support. A conversion pipeline (USD → glTF) or an alternative rendering library (e.g., Babylon.js with USD extensions, or NVIDIA Omniverse streaming) needs to be evaluated before implementation.

### 2. Scenario Editor
- Visualizes and edits YAML/JSON scenarios through the GUI.
- Displays initial snapshots and event schedules along a timeline.
- Sends saved scenarios to the Simulation Master via WebSocket.

### 3. Logic Editor (Behavior Tree Editor)
- Edits BT XML through the GUI.
- Builds Behavior Trees via drag-and-drop and converts them for use in SimPy or Gazebo nodes.
- Compatible with **Groot2** for advanced BT editing.

### 4. Log & Replay Viewer
- Loads and replays stored snapshots via WebSocket.
- Supports playback speed adjustment, event filtering, and statistical visualization.

---

## UI Design Highlights
- **Tabbed UI for each module** (3D View, Scenario, BT, Logs).
- **Drag-and-drop asset manipulation**.
- Emphasis on **real-time performance** by delivering snapshots via WebSocket with low latency.
- Designed for **ease of use by non-engineers**.

---

## Data Flow

1. Scenarios and Behavior Trees are edited in the GUI and sent to the server via WebSocket.  
2. The Web Server relays them to the Simulation Master via gRPC.  
3. During execution, delta snapshots are streamed from the Simulation Master via WebSocket and updated in the 3D view.  
4. During replay, stored snapshots are loaded and visualized in the GUI.
