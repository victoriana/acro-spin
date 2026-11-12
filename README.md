# Acro Spin

Agentic coaching application for acrobatic movement analysis. Designed to run on a Rust/ROS 2 sensor pipeline, with a LangGraph agent that decides each athlete's next drill from biomechanical scores and longitudinal skill history.

## Status

🚧 **Pre-alpha** — Agentic architecture scaffold in progress. Kinematic evaluators and simulated IMU data generator are the current focus. Hardware acquired (ESP32/BNO055 waist node); integration and validation pending. First deployment target is a **single waist-mounted node**.

## Architecture

| Layer | Tech | Role |
|---|---|---|
| `agent/` | Python + LangGraph | Coaching agent: tool-calling planner, feedback generation, session memory |
| `sensor-pipeline/` | Rust + ROS 2 (rclrs) | Real-time IMU ingestion & orientation estimation (consumes `ros2-rust-workspace`) |
| `sim/` | Python | Mock IMU publisher + gold-standard movement profiles (breadboard mode) |
| `orchestrator/` | Go + MCP | MCP tool server exposing metrics, history, and drill tools to the agent; session/state management |

## Agentic Design

Acro Spin separates **measurement** from **decision-making**. Deterministic code measures the movement; an LLM agent decides what to do about it.

**Deterministic layer (no LLM):** the Rust/ROS 2 pipeline and the kinematic evaluators compute every metric. The model never calculates or estimates a score.

**Agent layer (LLM with tools):** after each spin, a LangGraph agent chooses its own next action by calling tools exposed through the MCP orchestrator:

| Tool | Purpose |
|---|---|
| `get_session_metrics` | Read Tier 1 scores for the latest attempt |
| `get_history(metric, n)` | Read longitudinal trends across sessions |
| `compare_to_profile(profile)` | Compare against a gold-standard movement profile |
| `list_drills` | See the allowed drill progression |
| `set_next_drill(drill, focus_metric)` | Assign the next drill and the metric to prioritize |
| `request_retry` / `end_session` | Repeat the attempt or stop |

**What the agent decides:**
- Retry, regress, progress, or end the session
- Which metric to focus on, based on trends rather than just the last spin
- Which cue to give, and when to stop giving feedback and change the drill

The tool-calling loop continues until the agent chooses to finish. Routing is model-driven, not a fixed `if score > X` edge.

**Guardrails (outside the model):** hard limits on rotations per session, posture thresholds, and drill choices restricted to an allowed list. These are deterministic checks the agent cannot override, so the LLM reasons within safety bounds for physical activity.

**Agentic milestone:** the first agentic component is a planner node that reads metric history and selects the next drill and focus metric via validated structured output. It is built and tested against `sim/` before hardware integration.

## Evaluation Rubric

Acro Spin translates human movement intent into quantifiable metrics via the following proxy mappings. Metrics are tiered by sensor requirements so the rubric degrades gracefully to a single-node deployment.

### Tier 1 — Single waist node (first target)

| Metric Category | Human Goal | Computer Proxy (Sensor Data) |
| :--- | :--- | :--- |
| **Rotation Count** | "Hit the number of turns" | Yaw 360°-crossing count from waist sensor |
| **Velocity** | "Spin fast but controlled" | Peak Angular Velocity: Gyro Z-axis from waist sensor |
| **Stability** | "Don't wobble at end" | Settling Time: Gyro variance drop below threshold post-stop |
| **Posture** | "Stay upright" | Pitch/Roll Deviation: Max deviation from vertical axis during spin |
| **Smoothness** | "Even speed through the turn" | Std. deviation of yaw rate across each 360° segment |

### Tier 2 — Multi-node expansion (planned)

| Metric Category | Human Goal | Computer Proxy (Sensor Data) |
| :--- | :--- | :--- |
| **Alignment** | "Keep hips facing front" | Yaw Variance: Waist vs. Head Yaw deviation during setup |
| **Symmetry** | "Level arms" | Cross-Correlation: Phase difference between wrist sensors |

## Breadboard Workflow

The `sim/` layer lets you develop and test the full agentic loop without hardware:

1. Run `mock_imu.py` to publish fake quaternion streams with injected error patterns
2. LangGraph evaluator scores motion against `sim/profiles/*.json` using the rubric above
3. Agent calls tools, chooses the next drill and focus metric, and generates feedback
4. LangGraph checkpointer persists session state for longitudinal tracking

This mirrors the Lit-Flow-Flash read→write→evaluate pattern, adapted for biomechanics, with an agent layer that decides what happens next.

## Getting Started

```bash
git clone https://github.com/victoriana/acro-spin.git
cd acro-spin/sim && python mock_imu.py   # verify mock data stream
cd ../agent && pip install -r requirements.txt && python graph.py  # verify graph loads
```

## Related Projects

- **[ros2-rust-workspace](https://github.com/victoriana/ros2-rust-workspace)** — Core Rust IMU processing pipeline this app consumes. Contains reusable primitives (`esp32-bridge`, `imu-fusion`, `time-sync`) suitable for upstream contribution.

