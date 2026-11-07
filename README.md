# Acro Spin

Agentic coaching application for acrobatic movement analysis. Built on a high-performance Rust/ROS 2 sensor pipeline with a LangGraph feedback loop for real-time biomechanical evaluation and longitudinal skill tracking.

## Status

🚧 **Pre-alpha** — Agentic architecture scaffold in progress. Kinematic evaluators and simulated IMU data generator are the current focus. Hardware validation pending ESP32/BNO085 integration.

## Architecture

| Layer | Tech | Role |
|---|---|---|
| `agent/` | Python + LangGraph | Coaching brain: state routing, kinematic scoring, feedback generation, session memory |
| `sensor-pipeline/` | Rust + ROS 2 (rclrs) | Real-time IMU fusion & orientation estimation (consumes `ros2-rust-workspace`) |
| `sim/` | Python | Mock IMU publisher + gold-standard movement profiles (breadboard mode) |
| `orchestrator/` | Go + MCP | Session/state management between agent and sensors |

## Evaluation Rubric

Acro Spin translates human movement intent into quantifiable metrics via the following proxy mappings:

| Metric Category | Human Goal | Computer Proxy (Sensor Data) |
| :--- | :--- | :--- |
| **Alignment** | "Keep hips facing front" | Yaw Variance: Waist vs. Head Yaw deviation during setup |
| **Velocity** | "Spin fast but controlled" | Peak Angular Velocity: Gyro Z-axis from waist sensor |
| **Stability** | "Don't wobble at end" | Settling Time: Gyro variance drop below threshold post-stop |
| **Posture** | "Stay upright" | Pitch/Roll Deviation: Max deviation from vertical axis |
| **Symmetry** | "Level arms" | Cross-Correlation: Phase difference between wrist sensors |

## Breadboard Workflow

The `sim/` layer lets you develop and test the full agentic loop without hardware:

1. Run `mock_imu.py` to publish fake quaternion streams with injected error patterns
2. LangGraph evaluator scores motion against `sim/profiles/*.json` using the rubric above
3. Agent routes to coaching nodes and generates kinematic feedback
4. LangGraph checkpointer persists session state for longitudinal tracking

This mirrors the Lit-Flow-Flash read→write→evaluate pattern, adapted for biomechanics.

## Getting Started

```bash
git clone https://github.com/victoriana/acro-spin.git
cd acro-spin/sim && python mock_imu.py   # verify mock data stream
cd ../agent && pip install -r requirements.txt && python graph.py  # verify graph loads
```
## Related Projects

- **ros2-rust-workspace** — Core Rust IMU processing pipeline this app consumes. Contains reusable primitives (`esp32-bridge`, `imu-fusion`, `time-sync`) suitable for upstream contribution.

## License

MIT
