# Unified Simulation Orchestrator Development Plan

| Task | Start Date | End Date |
|------|------------|----------|
| Repo setup & directory structure | 2025-07-28 | 2025-08-04 |
| Build system (Python/C++/JS) & gRPC codegen | 2025-08-04 | 2025-08-14 |
| YAML/JSON/Asset schema definition | 2025-08-14 | 2025-08-19 |
| ZeroMQ messaging library (Master <-> Node) | 2025-08-21 | 2025-08-31 |
| Redis snapshot cache | 2025-08-31 | 2025-09-07 |
| gRPC API & ROS2 wrapper | 2025-09-07 | 2025-09-17 |
| Simulation Master core (distributed mode) | 2025-09-17 | 2025-10-01 |
| Single mode implementation | 2025-10-01 | 2025-10-06 |
| SimPy Node (BT + logic sim) | 2025-10-08 | 2025-10-22 |
| Gazebo Node (BT + physics sim) | 2025-10-22 | 2025-11-12 |
| Behavior Tree converter (SCXML <-> py_trees/BT.CPP) | 2025-11-12 | 2025-11-22 |
| Scenario parser & snapshot init | 2025-11-22 | 2025-11-29 |
| Web GUI (React + Three.js) core | 2025-11-29 | 2025-12-13 |
| Scenario & BT editor integration | 2025-12-13 | 2025-12-23 |
| Log/replay viewer | 2025-12-23 | 2025-12-30 |
| Unit tests (schemas, messaging, BT) | 2025-12-30 | 2026-01-06 |
| Integration tests (SimPy + Master) | 2026-01-06 | 2026-01-16 |
| Distributed test (multi-node) | 2026-01-16 | 2026-01-26 |
| CI/CD (GitHub Actions, Docker Compose) | 2026-01-26 | 2026-01-31 |
