# Mission Control

Mission Control observes and controls the CI/CD and deployment state of self-hosted services from one place, through a TUI for humans and a CLI for agents.

This glossary records the vocabulary as understood today. Two terms are marked `_Unsettled_`: they are placeholders standing in for a decision that has not been made, and they should not harden into the project's language by default.

## Language

### The product

**Mission Control**:
A TUI and CLI for observing and controlling the CI/CD and deployment state of self-hosted services from one place.
_Avoid_: dashboard, console, control plane

### The abstraction

**Domain**:
One of the three provider-agnostic capability areas the foundation abstracts: VCS, CI/CD, and Observability. Each domain defines an interface; providers implement it.
_Unsettled_: "Domain" is a working label. It collides with the design sense of the word (the problem space this project models), which is confusing in a repo that keeps a `CONTEXT.md`. The original phrasing was "general terms". Alternatives worth weighing: Capability, Concern, Facet, Surface.

**Provider**:
A concrete implementation of a single domain's interface. The first providers are GitLab (VCS), GitLab CI/CD (CI/CD), and Kubernetes (Observability).
_Avoid_: integration, backend, adapter, plugin, driver

**Foundation**:
The provider-agnostic core: the domain interfaces, the service and stage identity model, the TUI/CLI shell, and the AXI-compliant command surface. Everything a provider plugs into.
_Unsettled_: this layer has no agreed name. "Foundation" is a placeholder; naming it is open work.

### What is watched

**Service**:
The logical unit Mission Control monitors — one thing you ship, binding together a VCS repository, a CI/CD pipeline, and one or more deployments.
_Avoid_: app, application, project, repo, workload

**Stage**:
An environment a service is deployed to, such as dev, staging, or prod. Stages are cross-domain: a service's identity spans all of them, and each stage binds to its own provider instance context (a Kubernetes context, for the Observability domain).
_Avoid_: environment, env, tier, cluster, namespace

### The agent interface

**AXI** (Agent eXperience Interface):
The agent-ergonomic CLI design specification (https://axi.md/) that Mission Control's CLI conforms to fully. It governs how a CLI presents itself to an agent: token-efficient structured output, minimal default field sets, definitive empty states, structured errors, and contextual next-step hints.
_Avoid_: agent mode, machine output, JSON mode
