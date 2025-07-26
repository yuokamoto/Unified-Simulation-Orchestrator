# 9. Build Strategy and Directory Structure

## Purpose
This chapter defines the build and code generation strategy for the simulation framework,  
covering a multi-language environment (Python, C++, JavaScript) with an integrated build flow.  
Future integration into a **Bazel-based monorepo** is also considered.

---

## Build Strategy

### 1. Current Build Approach
- **Python**  
  - Uses `setuptools` and `pip` for packaging libraries and tools.  
  - Optionally supports `poetry` for simplified dependency management.
- **C++**  
  - Built with CMake.  
  - Responsible for simulation nodes (Gazebo, Genesis, Mujoco) and gRPC servers.
- **JavaScript (Node.js)**  
  - Uses `npm`/`yarn` for dependency management of the Web GUI and WebSocket server.

### 2. gRPC Code Generation
- Uses unified `.proto` definitions for all languages.  
- Generates code with `protoc` and language-specific plugins:
  - Python: `grpcio-tools`
  - C++: `grpc_cpp_plugin`
  - JavaScript: `grpc-tools`, `@grpc/proto-loader`
- Code generation is centrally managed with `scripts/gen_grpc.sh`.

### 3. Future Bazel Integration
- Transition components to Bazel build rules, enabling a monorepo structure.  
- Advantages of Bazel:
  - Unified management of multi-language dependencies.
  - CI/CD optimization via caching and incremental builds.
  - gRPC code generation integrated as Bazel build targets.

---

## Proposed Directory Structure

```
/project-root
  /proto                  # gRPC .proto files (shared across all languages)
  /scripts                # Build, code generation, and CI scripts
  /python
    /core                 # Simulation Master and SimPy nodes
    /tools                # Snapshot and scenario editing tools
  /cpp
    /nodes                # Gazebo/Genesis/Mujoco nodes
    /libs                 # Shared libraries (snapshot, BT, communication)
  /web
    /frontend             # React/Three.js frontend
    /server               # WebSocket/gRPC-Web server (Node.js)
  /config                 # Environment and build configuration files
  /tests                  # Integration and unit tests
  /docs                   # Documentation
```

---

## CI/CD
- Uses **GitHub Actions** or **GitLab CI**.  
- Defines build and test jobs per language.  
- Automatically runs gRPC code generation before builds.  
- If Bazel is adopted, CI/CD can be consolidated into a single job.

---

## Extension Points
- Full adoption of Bazel to enable multi-platform cross-compilation and caching.  
- Docker-based environment standardization, enabling containerized deployment of nodes and tools.
