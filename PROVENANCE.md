# Source Provenance

Audit date: 2026-10-07.

## EdgeCloudSim Baseline

- Upstream repository: https://github.com/CagataySonmez/EdgeCloudSim
- Matching release baseline: `v4.0`.
- Release commit: `0d253030b2161f377b9ae5ec259c8245d9769cbc`.
- Commit date: 2020-10-30.
- Release license: [GPL v3 text](https://github.com/CagataySonmez/EdgeCloudSim/blob/0d253030b2161f377b9ae5ec259c8245d9769cbc/LICENSE).
- Original source prefix: `src/edu/boun/edgecloudsim/`.
- Local source prefix: `src/main/java/edu/boun/edgecloudsim/`.

The user identifies this upstream repository as the source of much of the
Java framework. Comparison against upstream releases provides strong evidence
of a v4.0-era baseline: 30 of the 37 current Java files match v4.0 at their
mapped paths after CRLF/LF normalization; three mapped files differ; two
additional files are relocated copies from `sample_app4`; and two application
files have no counterpart at those mapped paths.

| Reference compared | Identical mapped Java files |
| --- | ---: |
| `v2.0` | 2 |
| `v3.0` | 4 |
| `v4.0` | 30 |
| Pre-import snapshot `f6d892d73119d5c35a854d25e3c6ac56d8137f4b` | 30 |

This identifies a matching reference, not a proven original checkout. The
later pre-import snapshot has the same mapped-file matches, so the source
could have been the release, a later checkout, or another copy of those files.
Do not label the exact original upstream commit as confirmed without an
archive, download record, preserved clone, or other evidence.

## Local History

The current framework paths first appear in local commit `6fe58fe` on
2025-04-27, with message `all new stuff`. The project's Git history starts on
2025-04-15 and contains earlier development entries, but the framework import
does not identify an upstream SHA. Import dates establish when files were
recorded locally; they do not establish when or by whom each modification was
originally made.

## Unchanged Framework Files

The following paths, relative to `src/main/java/edu/boun/edgecloudsim/`, match
the v4.0 files after line-ending normalization:

```text
cloud_server/CloudServerManager.java
cloud_server/CloudVM.java
cloud_server/CloudVmAllocationPolicy_Custom.java
cloud_server/DefaultCloudServerManager.java
core/ScenarioFactory.java
core/SimManager.java
core/SimSettings.java
edge_client/CpuUtilizationModel_Custom.java
edge_client/MobileDeviceManager.java
edge_client/mobile_processing_unit/DefaultMobileServerManager.java
edge_client/mobile_processing_unit/MobileHost.java
edge_client/mobile_processing_unit/MobileServerManager.java
edge_client/mobile_processing_unit/MobileVM.java
edge_client/mobile_processing_unit/MobileVmAllocationPolicy_Custom.java
edge_orchestrator/BasicEdgeOrchestrator.java
edge_orchestrator/EdgeOrchestrator.java
edge_server/DefaultEdgeServerManager.java
edge_server/EdgeHost.java
edge_server/EdgeServerManager.java
edge_server/EdgeVM.java
edge_server/EdgeVmAllocationPolicy_Custom.java
mobility/MobilityModel.java
mobility/NomadicMobility.java
network/MM1Queue.java
network/NetworkModel.java
task_generator/IdleActiveLoadGenerator.java
task_generator/LoadGeneratorModel.java
utils/Location.java
utils/SimUtils.java
utils/TaskProperty.java
```

## Modified and Relocated Framework Files

The following changes are present in the audited local tree relative to
v4.0. These files were first recorded at their current paths on 2025-04-27;
that is the recorded import date, not a claimed original modification date.

| Local path relative to the Java package root | Changes from v4.0 |
| --- | --- |
| `edge_client/Task.java` | Adds a task deadline field, constructor argument, and accessor. |
| `edge_client/DefaultMobileDeviceManager.java` | Computes deadlines using edge VM capacity and passes them into tasks. |
| `utils/SimLogger.java` | Adds the scheduler-facing `simLog` logging method. |
| `edge_client/FuzzyExperimentalNetworkModel.java` | Relocates `applications/sample_app4/FuzzyExperimentalNetworkModel.java`; changes the package declaration. |
| `edge_client/FuzzyMobileDeviceManager.java` | Relocates the corresponding `sample_app4` file; changes the package and supplies a deadline argument to the task constructor. |

## Application Additions

`MainSimulator.java` and `applications/RLTaskScheduler.java` have no counterpart
at their mapped paths in v4.0. They provide the project's simulation entry
point and RL orchestration. Absence at those paths does not prove that every
line was independently authored; contributor and external-source attribution
still need confirmation.

## Python Training Paths

| Path | Implementation and relationship to Java |
| --- | --- |
| Independent training | `RLPython/cloud_egde_agent.py` defines `CloudEdgeEnv` and `CloudEdgeSimulator`; `train_dqn()` and `train_ppo()` generate experiences in this synthetic environment without Java. `tester.py` exercises this path. |
| Java-dependent training | `RLTaskScheduler` records Java simulation experiences in `rl_experiences.csv`. Its `trainModel()` method expects a Python `train_rl_model.py` to consume those experiences. That script is not present, so this path is incomplete. |
| Java-dependent inference | `RLPython/rl_api_server.py` serves predictions to the Java scheduler. Serving predictions does not itself train a model; state-format and routing mismatches remain. |

Both training paths are included in the GPL v3 policy for original project
code. Their runtime independence does not establish authorship, and it does
not require separate licensing when their copyright holders choose the same
license. The provenance of the Python implementation must still be confirmed
with its contributors; no specific external source is established here.

## Bundled Binaries

All six `lib/` JARs match their v4.0 counterparts byte-for-byte:

```text
lib/cloudsim-4.0.jar
lib/colt.jar
lib/commons-math3-3.6.1.jar
lib/jFuzzyLogic_v3.0.jar
lib/mtj-1.0.4.jar
lib/weka.jar
```

This is strong evidence of their upstream source. It does not replace their
individual license notices or required corresponding source. Checkpoint
provenance in `models/` and `RLPython/models/` has not been established by this
source comparison.

## Reproducing and Completing the Investigation

Clone upstream into a separate directory and inspect the release history:

```bash
git clone https://github.com/CagataySonmez/EdgeCloudSim.git ../EdgeCloudSim-upstream
git -C ../EdgeCloudSim-upstream rev-list -n 1 v4.0
git -C ../EdgeCloudSim-upstream log -1 --format=fuller v4.0
git log --follow -- src/main/java/edu/boun/edgecloudsim/core/SimManager.java
```

Compare a mapped file from the project root with upstream's release version:

```bash
git -C ../EdgeCloudSim-upstream show \
  v4.0:src/edu/boun/edgecloudsim/core/SimManager.java \
  | diff --strip-trailing-cr - src/main/java/edu/boun/edgecloudsim/core/SimManager.java
```

Repeat for other framework files, accounting for the `sample_app4`
relocations. The audit compared decoded file bytes after normalizing CRLF to
LF; it did not remove other whitespace or comments.

To establish the actual original checkout, ask the contributor who imported
the framework for the downloaded archive name, upstream clone or SHA, fork
URL, or original source directory. Record any evidence here. Confirm Python
sources and contributor permissions, collect dependency licensing material,
and document model creation settings and training inputs before calling the
provenance complete.
