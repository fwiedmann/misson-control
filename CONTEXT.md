# Mission Control

Mission Control observes and controls the CI/CD and deployment state of self-hosted services from one place, through a TUI for humans and a CLI for agents.

This glossary records the vocabulary as understood today.

## Language

### The product

**Mission Control**:
A TUI and CLI for observing and controlling the CI/CD and deployment state of self-hosted services from one place.
_Avoid_: dashboard, console, control plane

### The abstraction

**Capability**:
One of the three provider-agnostic areas Mission Control abstracts: VCS, CI/CD, and Runtime. Each capability defines an interface, and providers implement it.
_Typed as_: `vcs`, `ci`, `runtime` — note that **CI/CD** is deliberately typed `ci`
_Avoid_: domain, concern, facet, surface, area

**Runtime**:
The capability covering a service's running deployments: what is deployed to a stage, its health, and acting on it (restart, stop).
_Avoid_: observability, ops, infra, cluster

**Feature**:
A single operation within a capability that a given provider may or may not support, such as watching a pipeline for live updates.
_Avoid_: capability (in this fine-grained sense), support flag

**Provider**:
A concrete implementation of a single capability's interface. The first providers are GitLab (VCS), GitLab CI/CD (CI/CD), and Kubernetes (Runtime).
_Avoid_: integration, backend, adapter, plugin, driver

**Core**:
The provider-agnostic centre: the capability interfaces, the service and stage identity model, the TUI/CLI shell, and the AXI-compliant command surface. Everything a provider plugs into.
_Avoid_: foundation, kernel, chassis, framework, platform

### What is watched

**Service**:
The logical unit Mission Control monitors — one thing you ship, binding together a VCS repository, a CI/CD pipeline, and one or more deployments.
_Avoid_: app, application, project, repo, workload

**Stage**:
An environment a service is deployed to, such as dev, staging, or prod. Stages are cross-capability: a service's identity spans all of them, and each stage binds to its own provider instance context (a Kubernetes context, for the Runtime capability).
_Avoid_: environment, env, tier, cluster, namespace

### The agent interface

**AXI** (Agent eXperience Interface):
The agent-ergonomic CLI design specification (https://axi.md/) that Mission Control's CLI conforms to fully. It governs how a CLI presents itself to an agent: token-efficient structured output, minimal default field sets, definitive empty states, structured errors, and contextual next-step hints.
_Avoid_: agent mode, machine output, JSON mode
