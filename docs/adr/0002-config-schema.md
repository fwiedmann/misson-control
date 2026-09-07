---
status: accepted
---

# One YAML config file, with a global stage axis and provider-scoped blocks

Services are declared by hand in a single YAML file at `~/.config/mission-control/config.yaml` (honouring `$XDG_CONFIG_HOME`, overridable by `--config` and `$MISSION_CONTROL_CONFIG`). Stages are a **global axis** that services bind to by name, and every provider's settings live in a block keyed **capability, then provider** — opaque to the core and strictly decoded.

```yaml
version: 1

stages:                          # global axis; declaration order is display order
  - name: dev
    runtime:
      kubernetes:
        context: homelab-dev
  - name: prod
    runtime:
      kubernetes:
        context: homelab-prod

services:
  - name: api                    # unique, the CLI addressing token
    stages: [dev, prod]          # subset of the global stages
    vcs:
      gitlab:
        project: group/api
    ci:
      gitlab:
        project: group/api
    runtime:
      kubernetes:
        namespace: api           # stage-invariant, given one stage per cluster
        workload: api
```

## Why stages are global rather than per-service

Two independent arguments arrive at the same answer.

**The stages overview is a matrix** — services down, stages across — and a matrix needs a shared axis. If each service declares its own stage names, `staging` / `stage` / `stg` silently become three columns, and the "everything in one place" promise dies in the display layer.

**Stages are expensive objects.** Runtime access is one clientset and one `SharedInformerFactory` per Kubernetes context. A global axis yields exactly N factories for N contexts; per-service stages would need dedup logic to avoid 30 services × 3 stages = 90 factories.

This works because **each stage is its own cluster**, which makes a service's namespace and workload name stage-invariant. Should two stages ever share a cluster and be separated *by* namespace, a per-stage override becomes necessary — a purely additive change, which is why it is deferred rather than built.

## Why blocks are keyed capability, then provider

`runtime.kubernetes.*`, not `kubernetes.*`. This is settled by a fact rather than taste: **GitLab is two providers**, implementing both VCS and CI/CD. A provider-only key would force one `gitlab:` block to serve both capabilities and require disambiguation rules immediately, where `vcs.gitlab` and `ci.gitlab` are separate and may legitimately carry different project paths — pipelines in a separate deploy repo is a normal self-hosted shape.

It also **quarantines provider vocabulary**. `namespace` and `workload` are Kubernetes words that live inside a Kubernetes block and never become core terms; `CONTEXT.md` needed no change to accommodate this schema, and already bans *namespace* as a synonym for **Stage**.

## Consequences

- **The core cannot validate provider blocks.** It parses the skeleton — name, stages, which capabilities are present — and hands each block to its provider as raw YAML. Error quality inside those blocks is each provider's job, and the core must never import provider config types. The registration and construction contract this implies is [issue #15](https://github.com/fwiedmann/misson-control/issues/15).
- **Decoding is strict**: unknown keys are errors. In a hand-written file, a typo'd key that silently does nothing is the worst available failure mode.
- **No cwd search for the config file.** Behaviour that depends on where the user was standing is what AXI's definitive-behaviour rules exist to prevent; an agent must be able to predict which config it got.
- **No dotenv loading.** `KUBECONFIG` is read as standard with `MC_KUBECONFIG` overriding, and GitLab takes `MC_GITLAB_TOKEN` — ambient environment only. Same determinism argument as the config path, and `direnv` does the job better and per-directory. If a `.env` is ever wanted it should be an explicit `--env-file` flag, never a search.
- **No secrets in the file**, which is diffable and will end up in somebody's dotfiles repo. A schema that *permits* an inline token guarantees one eventually gets committed.
- **No cross-capability defaulting.** `ci.gitlab.project` does not fall back to `vcs.gitlab.project`; defaulting couples blocks the capability model exists to separate, and renaming the `vcs` block would break the CI/CD binding elsewhere.
- **VCS is required, plus at least one of CI/CD or Runtime.** A VCS-only entry is a bookmark, not a service, and would give the overview rows that can never show anything.
- **No slot is reserved for image-tag→source resolution** ([issue #11](https://github.com/fwiedmann/misson-control/issues/11)). Because provider blocks are opaque, that decision can add keys later as a purely additive change; reserving one now would mean guessing the shape of an unmade decision.

## Considered options

- **Per-service stages** — rejected by both arguments above.
- **Hybrid (global stage definitions, per-service per-stage overrides)** — rejected for now only because one stage per cluster makes the override unnecessary. It remains the additive escape hatch if that ever stops being true.
- **TOML** — genuinely viable once the global stage axis made the file shallow, and safer for agents to edit. Rejected because the audience already lives in YAML (kubeconfig, GitLab CI, k8s manifests) and this file names kubeconfig contexts, keeping the mental model continuous. YAML's implicit-typing hazard barely applies: every field is a string or a list of strings.
- **JSON** — no comments in a hand-written file.
- **Deriving the service name from the VCS project path** — rejected: GitLab paths contain slashes and cannot be shell tokens, and no single provider's name is authoritative when the repo is `payment-service`, the Deployment is `payments`, and the pipeline is neither.

Full reasoning: [issue #5](https://github.com/fwiedmann/misson-control/issues/5).
