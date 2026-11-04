# Acro Spin

Agentic coaching application for acrobatic movement analysis. Built on a Rust/ROS 2 sensor pipeline with a LangGraph feedback loop.

## Status

🚧 **Pre-alpha** — Agentic architecture scaffold in progress. Kinematic evaluators and simulated IMU data generator are the current focus.

## Architecture

| Layer | Tech | Role |
|---|---|---|
| `agent/` | Python + LangGraph | Coaching brain: state routing, kinematic scoring, feedback generation |
| `sensor-pipeline/` | Rust + ROS 2 (rclrs) | Real-time IMU fusion & orientation estimation |
| `sim/` | Python | Mock IMU publisher + gold-standard movement profiles (breadboard mode) |
| `orchestrator/` | Go + MCP | Session/state management between agent and sensors |

## Breadboard Workflow

The `sim/` layer lets you develop and test the full agentic loop without hardware:

1. Run `mock_imu.py` to publish fake quaternion streams with injected error patterns
2. LangGraph evaluator scores motion against `sim/profiles/*.json`
3. Agent routes to coaching nodes and generates kinematic feedback

This mirrors the Lit-Flow-Flash read→write→evaluate pattern, adapted for biomechanics.

## Getting Started

```bash
git clone https://github.com/victoriana/acro-spin.git
cd acro-spin/sim && python mock_imu.py   # verify mock data stream
cd ../agent && pip install -r requirements.txt && python graph.py  # verify graph loads
```

## Related Projects

ros2-rust-workspace — Core Rust IMU processing pipeline this app consumes

## License
MIT

