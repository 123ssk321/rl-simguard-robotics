# Research basis

Source: RL project research, 16 October 2026. Extract preserved from the report; references were not revalidated during delivery publication on 24 October 2026. Product potential is a hypothesis. The execution plan is the narrower delivery contract if timing or scope differs.

# Project 2 — SimGuard Robotics

## Identity and problem

**One sentence:** A local policy qualification tool that runs a simulated robot policy through repeatable regression and perturbation scenarios and produces a release decision with evidence.

Robotics developers need to catch a regression when changing a policy, environment, or configuration. A peak score on a nominal simulation can miss sensitivity to mass, friction, sensor noise, delay, initial state, or contact conditions. SimGuard provides a versioned scenario suite and comparisons to a previous policy and a conventional controller.

**Initial user hypothesis:** robotics developers, small robotics teams, and research/software labs that already use simulation. The public prototype can be built without interviews or robot access; validating the product would later require feedback from a robotics team.

**Why now and research foundation:** MuJoCo is open source and Gymnasium provides low-dimensional continuous-control environments. Gymnasium-Robotics includes Fetch manipulation tasks; robosuite provides robot-learning simulations and a domain-randomization wrapper. A 2017 domain-randomization paper showed a specific sim-to-real visual localization result, but that is not evidence that any simulated RL policy is safe or transferable. [Tobin et al. (2017)](https://arxiv.org/abs/1703.06907); [robosuite documentation](https://robosuite.ai/docs/); [Gymnasium-Robotics tasks](https://robotics.farama.org/).

**Skills demonstrated:** continuous-control policy interfaces, simulator integration, domain randomization, robustness and regression testing, uncertainty-aware release gates, CI integration, artifact/version management, visualization, and safety-conscious scope.

## Product/system concept

Start with a low-dimensional reach or push task; do not begin with camera-based manipulation, humanoids, a robot arm purchase, or a ROS/hardware deployment. A built-in Gymnasium environment can prove the runner architecture; a later adapter can wrap a more representative robosuite task.

**Workflow:** developer selects a policy artifact and pinned environment; runs a standard suite; optionally adds generated perturbations within documented bounds; compares policy to prior release and baseline; receives pass/fail/inconclusive with trial count, uncertainty, violations, and representative failure clips. A pass is a software regression signal only, never a safety certificate.

**Architecture:** policy adapter contract → pinned Gymnasium/MuJoCo or robosuite environment → scenario manifests (seed, initial state, parameter ranges, disturbance schedule) → bounded episode runner → metric and invariant evaluator → HTML/JSON report and CLI exit code. Record hashes for policy, config, environment package, and scenario set. Run a fixed nominal suite plus generated perturbation suites. Include an `inconclusive` gate when trials are insufficient or confidence bounds cross thresholds.

**Data:** no manually assembled datasets. Generate scenarios from the simulator with deterministic seeds and declared distributions. Store scenario specifications, not giant precomputed observations; regenerate on demand. The policy checkpoint is an artifact, not an externally sourced dataset. Pin license terms for robot models/assets and simulator dependencies; do not bundle assets whose redistribution rights are unclear.

## Evaluation and success criteria

- Compare a trained policy, prior policy, and built-in/controller baseline on paired scenario seeds.
- Measure task completion, constraint/collision events available in the environment, joint/action limits, completion time, action smoothness, inference latency, and performance distribution—not just average reward.
- Create fault injection cases with a deliberately degraded checkpoint/config so the tool's regression gate is shown to catch known changes. Test if and when it produces false pass/fail behavior under scenario sampling.
- Include nominal, shifted-parameter, and compound-stress suites. Report sample counts and uncertainty.
- Public success: a clean install and reproducible regression report, a video of success and failure traces, a documented scenario schema, tests around simulator/API contract, and limitations prominently saying “simulation-only; no hardware safety or transfer claim.”

## Delivery plan (16 hours/week)

| Phase | Scope and exit gate |
|---|---|
| **Months 1–2** | Pin an environment; inspect observation/action semantics; establish a deterministic baseline; implement policy interface, seed control, and scenario manifest. Keep one task only. |
| **Months 3–4: beta** | Publish only once the runner produces a reproducible report, catches an intentionally degraded policy, and provides a useful failure artifact. A first beta in month 4 is realistic. |
| **Months 5–7: hardening** | Stabilize plugin/API boundary, handle timeouts/errors, add report schemas, policy/environment hashes, CI mode, resource caps, and regression thresholds with confidence intervals. |
| **Months 8–10: stable local release** | Add a second task or robot embodiment only after the runner is stable; improve installation and docs; publish versioned artifacts and release notes. |
| **Months 11–12** | Seek voluntary external review or a design partner; consider ROS 2 or hardware adapters only with suitable access and safety review. Do not block the software product on hardware. |

**Difficulty/dependencies:** medium-high due to simulator setup, scenario semantics, and physics interpretation. The first state-based task fits laptop CPU and modest GPU. Gymnasium-Robotics officially supports Linux/macOS and accepts but does not officially support Windows PRs per its repository README, so on this Windows machine prefer a verified WSL/Linux workflow or use a Gymnasium MuJoCo environment first; check current compatibility before choosing the stack. [Gymnasium-Robotics README](https://github.com/Farama-Foundation/Gymnasium-Robotics).

## Career and product path

**Artifacts:** versioned CLI, scenario schemas, sample trained and deliberately degraded policy, regression reports, CI example, failure videos, test suite, architecture/security/limitations docs, and release package.

**CV bullet template:** “Implemented a local robot-policy qualification runner for [environment/task], executing [N] paired nominal and perturbed scenarios; detected [N] seeded regressions and reported [metrics] with [uncertainty method], packaged as a CI-compatible CLI.”

**Potential buyers:** small robotics companies, robotics SDK/simulation providers, integrators, and teams building manipulation systems. Potential value is catching changes earlier and creating a shared test record. Possible sustainability models are open-core local runner plus paid integration/support, or a private hosted team report product if users request it. Commercial potential is **moderate/uncertain**: real teams may already have internal test tooling, and adoption depends on simulator/controller integration. Partnership path is later collaboration with a robotics team or open-source maintainer; an unsolicited public demo is enough to initiate technical discussion without claiming endorsement.

**Risks:** missed rare failures, simulator bugs, incompatible physics versions, misleading pass rates, licenses of robot assets, and false interpretation as safety proof. Use only generated simulation trials, bounds and hashes; show raw results and uncertainty; avoid physical actuation. Worth pursuing if robotics remains motivating after the first environment and the user values tooling and reliability as much as policy training.
