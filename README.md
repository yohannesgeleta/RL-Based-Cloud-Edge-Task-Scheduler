# RL-Based Cloud-Edge Task Scheduler

A research prototype for reinforcement-learning-based task placement across edge and cloud computing resources. The project combines a Java EdgeCloudSim simulation with Python implementations of Deep Q-Network (DQN) and Proximal Policy Optimization (PPO).

The objective is to explore how scheduling decisions affect processing delay, estimated energy consumption, and task deadline success.

> **Project status:** Experimental. Python training and evaluation use a standalone synthetic environment. The Java-to-Python integration is incomplete; see [Known Limitations](#known-limitations) before running the full simulation.

## Overview

Cloud-edge systems must decide whether to execute tasks close to the originating device or transfer them to cloud resources. This decision depends on resource utilization, queued work, network conditions, and task requirements.

This repository provides:

- A Java simulation of mobile devices, edge and cloud servers, virtual machines, mobility, and network delays.
- A Python environment that generates tasks and models changing CPU loads, queues, and bandwidth.
- A dueling Double DQN agent with prioritized experience replay and epsilon-greedy exploration.
- A PPO agent with an actor-critic network and clipped policy updates.
- Training, evaluation, checkpoint saving, and performance plotting utilities.
- A Flask prediction API intended to connect trained policies to the Java scheduler.
- A weighted heuristic scheduler for manual selection and prediction fallback.

The environment uses approximate time and energy formulas. Its outputs are simulated metrics, not measurements from deployed hardware.

## Architecture

| Component | Responsibility |
| --- | --- |
| `MainSimulator.java` | Loads settings, constructs simulation components, and starts CloudSim. |
| `RLTaskScheduler.java` | Builds state vectors, selects destinations, estimates rewards, and records experiences. |
| `RLPython/cloud_egde_agent.py` | Defines the standalone environment and training/evaluation workflows. |
| `RLPython/dqn.py` | Implements the DQN network and learning agent. |
| `RLPython/ppo.py` | Implements the PPO network and learning agent. |
| `RLPython/experience_replay.py` | Stores and samples prioritized DQN experiences. |
| `RLPython/rl_api_server.py` | Serves model predictions over HTTP. |
| `RLPython/tester.py` | Trains both algorithms and generates evaluation plots. |

In the Python environment, an action selects an edge device or cloud server. Each step estimates execution time and energy, checks the task deadline, updates resource conditions, and generates the next task. The reward penalizes time and energy consumption and rewards meeting deadlines.

The intended Java integration sends a state vector to `POST /predict` and uses the returned action to select a VM. This integration currently requires corrections to routing and state representation.

## Repository Structure

```text
.
|-- src/main/java/edu/boun/edgecloudsim/
|   |-- MainSimulator.java
|   |-- applications/RLTaskScheduler.java
|   |-- core/
|   |-- edge_client/
|   |-- edge_server/
|   |-- cloud_server/
|   |-- network/
|   |-- mobility/
|   |-- task_generator/
|   `-- utils/
|-- RLPython/
|   |-- cloud_egde_agent.py
|   |-- dqn.py
|   |-- ppo.py
|   |-- experience_replay.py
|   |-- hyperparameters.yaml
|   |-- rl_api_server.py
|   |-- tester.py
|   |-- evaluate.py
|   |-- early_train_model.py
|   `-- models/
|-- models/
|-- lib/
|-- applications.xml
|-- edge_devices.xml
|-- MyScenario_networkTopology.xml
|-- rl_config.properties
`-- pom.xml
```

The core Python module is currently named `cloud_egde_agent.py`, including the spelling shown above. The two `models/` directories contain different checkpoint files; they should not be treated as interchangeable copies.

## Prerequisites

- Python 3.11 for the dependency baseline in `requirements.txt`.
- Python packages: `torch`, `numpy`, `matplotlib`, `PyYAML`, and `Flask`.
- A Java Development Kit compatible with the Java 8 source/target settings in `pom.xml`.
- Apache Maven for building the Java application.

Direct Python dependency versions are pinned in [requirements.txt](requirements.txt). These are not a complete lock of transitive dependencies. The training dependencies match the local Python environment; a clean installation including Flask has not yet been validated.

## Python Setup

Run these commands from the repository root:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

On Windows, activate the environment with `.venv\Scripts\activate` instead.

Run the Python workflows from `RLPython/` so imports, configuration paths, and relative checkpoint paths resolve consistently:

```bash
cd RLPython
```

### Train and Evaluate

For a small experiment, open an interactive Python session with `python` and run:

```python
from cloud_egde_agent import CloudEdgeSimulator

simulator = CloudEdgeSimulator(config_path="hyperparameters.yaml")
simulator.train_dqn(episodes=10, save_path="models/dqn_demo.pt")
simulator.train_ppo(episodes=10, save_path="models/ppo_demo.pt")

dqn_metrics = simulator.evaluate_agent(agent_type="dqn", episodes=10)
ppo_metrics = simulator.evaluate_agent(agent_type="ppo", episodes=10)
```

Short runs are useful for checking the workflow; they do not establish algorithm performance. Training saves final checkpoints with an `_final.pt` suffix. Periodic best-model checkpoints depend on the configured save frequency.

For the existing training-and-plotting workflow:

```bash
python tester.py
```

This script trains each algorithm for 500 episodes and evaluates each for 200 episodes. It writes checkpoints to `RLPython/models/` and saves plots in the working directory. Existing checkpoint names may be overwritten. Plot windows are interactive; use `MPLBACKEND=Agg` when running without a graphical display.

### Prediction API

The API exposes:

| Endpoint | Method | Purpose |
| --- | --- | --- |
| `/predict` | `POST` | Accepts `{"state": [...]}` and returns `{"action": <integer>}`. |
| `/health` | `GET` | Returns the configured algorithm and model path when model loading succeeds. |

The standalone Python environment defaults to three edge devices and two cloud servers, producing 14 state features and five actions. For a DQN checkpoint trained with those defaults, start the API from `RLPython/` with:

```bash
RL_MODEL_PATH=models/dqn_demo_final.pt \
RL_ALGORITHM=DQN \
STATE_DIM=14 \
ACTION_DIM=5 \
python rl_api_server.py
```

The server listens on port `5000`. Set dimensions to match the checkpoint used. The Java scheduler uses a different state format, so the example above is for standalone Python inference, not a working Java integration.

## Java Simulation

From the repository root, the configured build command is:

```bash
mvn package
```

The Maven configuration declares CloudSim `3.0.3` and creates an executable JAR with dependencies. The repository also contains a separate CloudSim `4.0` JAR under `lib/`; Maven does not automatically use that directory.

After a successful build, the intended launch command is:

```bash
java -jar target/edgecloudsim-rl-1.0-SNAPSHOT-jar-with-dependencies.jar \
  rl_config.properties edge_devices.xml applications.xml sim_results
```

The arguments are the simulation settings, edge-device definitions, application definitions, and output directory. Omitting them uses the paths shown above. File logging clears the selected output folder before a run, and training mode recreates `rl_experiences.csv`.

The current routing mismatch can terminate the simulation before a complete run. A fresh-clone Maven build has not been verified as part of this documentation update.

## Configuration

| File | Settings |
| --- | --- |
| `rl_config.properties` | Java scheduling mode, exploration, resource sizes, network parameters, simulation timing, and logging. |
| `applications.xml` | Task arrivals, active/idle periods, data sizes, task lengths, and utilization requirements. |
| `edge_devices.xml` | Datacenter locations, host resources, and VM definitions. |
| `RLPython/hyperparameters.yaml` | Replay capacity, learning rates, discount factor, exploration decay, and PPO update settings. |

The checked-in Java configuration enables training, disables pretrained-model inference, and sets exploration to `1.0`. Consequently, scheduling actions are selected randomly. Python training has its own exploration schedule and configuration.

API configuration is separate from Java properties and uses `RL_MODEL_PATH`, `RL_ALGORITHM`, `STATE_DIM`, and `ACTION_DIM` environment variables.

## Outputs

- **Python metrics:** Episode reward, deadline success rate, average processing time, and average estimated energy per task.
- **Model checkpoints:** DQN and PPO `.pt` files, including final training checkpoints.
- **Plots:** Reward, processing time, deadline success, energy, and learning/loss curves from `tester.py`.
- **Java results:** Simulation logs under the chosen output directory and scheduling experiences in `rl_experiences.csv` when training is enabled.

No benchmark results or performance advantage are claimed by this README. Comparisons require controlled experiments with documented configurations, seeds, and repeated runs.

## Known Limitations

- The Java scheduler returns a VM index where the mobile-device manager expects an edge/cloud destination identifier.
- Java and standalone Python use different state dimensions, feature ordering, and task representations. Matching array dimensions alone does not make their policies compatible.
- Java's `trainModel()` references `train_rl_model.py`, which is not included. Automatic training from Java-collected experiences is unfinished.
- Java experiences record an immediate next-state snapshot before execution, and scheduling rewards use estimated outcomes.
- The Java scheduler's energy metric is not accumulated, so its reported average energy is not a valid comparison measure.
- `evaluate.py` currently has its main workflow enclosed in a multiline string; `early_train_model.py` contains commented examples. Use the Python methods or `tester.py` instead.
- The API reloads its model before every request and uses Flask's development server.
- There is no automated test suite, CI workflow, or complete Python dependency lockfile in the repository.

## Attribution and Licensing

Original project contributions are licensed under the **GNU General Public License, version 3 only (`GPL-3.0-only`)**. See [LICENSE](LICENSE) for the full text and [LICENSING.md](LICENSING.md) for scope, third-party exclusions, and remaining redistribution checks. This policy covers original Java contributions and both standalone and Java-dependent Python training code.

The Java framework derives from [EdgeCloudSim](https://github.com/CagataySonmez/EdgeCloudSim), with existing copyright notices attributing it to Bogazici University. Thirty Java files match EdgeCloudSim v4.0 after line-ending normalization, and all six bundled JARs match that release byte-for-byte. This establishes a matching baseline, not the exact checkout originally used.

See [PROVENANCE.md](PROVENANCE.md) for the upstream commit, modified and relocated files, the independent and dependent training paths, and reproduction instructions. Inherited code and dependencies retain their applicable licenses. Contributor permissions, dependency redistribution material, and model/data provenance still need confirmation.

## Contributing

For proposed changes, describe the problem, include the relevant configuration and validation steps, and distinguish standalone Python behavior from Java integration behavior. Algorithm comparisons should include reproducible experiment settings and disclose limitations.
