# 6. Logging / Replay Specification

## Purpose
The **Logging / Replay functionality** of this framework records simulation states and events during execution,
enabling **offline analysis, visualization, debugging**, and **reproduction testing** afterward.

---

## Log Structure

### Basic Units of Logging
1. **Full Snapshot**
   - The complete world state at a specific time `t` (positions, velocities, internal states of all assets).
   - Saved periodically (e.g., every second) and used as replay anchors.

2. **Delta Snapshot**
   - Records only changes since the previous step (position and state updates).
   - Lightweight format, used to interpolate between full snapshots.

3. **Event Log**
   - Records discrete events such as task completions, BT state transitions, or error occurrences.

---

## Log Storage
- **Redis Cache**
  - Used for real-time synchronization in distributed mode.
  - Stores full snapshots as recovery points for node failures.

- **Persistent Storage**
  - Sequentially stores full snapshots, delta snapshots, and event logs as files.
  - Stored in JSON or binary format (selectable for optimization).

---

## Replay Specification

1. **Load a full snapshot** to initialize the world state at a specific time.
2. **Apply delta snapshots sequentially** to reproduce the original simulation progression.
3. **Adjustable playback speed** (1x, slow motion, fast-forward).
4. **Event filtering** (display only specific tasks or errors).
5. **Support for visual playback via GUI** or **statistical analysis via CLI**.

---

## Usage

### CLI Tools
- `log list`: Display a list of available logs.
- `log replay <log_id>`: Replay a log (text-based or simple visual mode).
- `log export <log_id> --format=csv`: Export logs for external analysis.

### GUI Tools
- Playback of snapshots along a timeline.
- Highlight changes (deltas) visually.
- Event filtering and display of statistical dashboards.

---

## Implementation Notes
- Adjust full snapshot intervals to **optimize replay performance and storage usage**.
- Deltas are **detected and compressed per asset** for efficiency.
- The log format is designed to **remain compatible with scenario and snapshot specifications**.
