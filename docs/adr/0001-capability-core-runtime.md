---
status: accepted
---

# Capability, Core, and Runtime, with the capability set closed at three

Mission Control abstracts three provider-agnostic areas behind interfaces that providers implement. Those areas are called **capabilities** (VCS, CI/CD, Runtime), the provider-agnostic centre that defines them is the **core**, and the set is **closed at three** — deliberately, and on a constraint rather than an assumption. `Capability` and `Core` replace the `Domain` and `Foundation` placeholders the glossary carried; `Runtime` replaces `Observability`.

## Why Runtime, not Observability

This is a correction, not a rename. Mutations are in v1: the interface for a running deployment carries `Restart` and `Stop` alongside status and logs. An interface named `Observability` that mutates the thing it observes is lying about itself, and the name would have quietly discouraged putting control operations where they belong. `Runtime` names the running system, which is equally at home observing it and acting on it.

## Why the set stays at three

A container registry is the obvious fourth capability, and the pressure point is resolving a deployed image tag back to a VCS tag. Reading OCI labels (`org.opencontainers.image.revision`) off an image means an authenticated registry call, and a registry is an artifact store that none of VCS, CI/CD, or Runtime naturally owns.

**So image-to-source resolution must never require a container registry.** It resolves through the tag string itself, through Kubernetes annotations written at deploy time, or by asking CI/CD which pipeline produced the image. This constraint is what holds the set at three; if it is ever broken, the closed set goes with it.

## Considered options

- **No collective noun at all** — with the set closed at three, flat `internal/vcs`, `internal/ci`, `internal/runtime` packages and three glossary entries were genuinely viable. Rejected because the spec's central sentence generalises over all three ("a provider implements one capability for one service"), and writing it three times is a worse trade than teaching one word.
- **Port** (hexagonal, ports-and-adapters) — the most exact available lineage for what these actually are. Rejected because the glossary already bans *adapter* under **Provider**, so adopting *port* imports half a vocabulary deliberately rejected; it also collides with network port.
- **Surface** — rejected outright; "command surface" is already in use for the AXI CLI.
- **Concern**, **Facet** — rejected as soft and as evocative-of-nothing respectively.
- **Kernel**, **Chassis** for the core — both name "the thing everything plugs into" better than `Core` does. Rejected because once `Capability` and `Provider` exist as terms, that idea is already carried by *"a provider implements a capability"*; `Core` only needs to mean *not a provider*.

## Consequences

- **`Feature` is reserved** for the fine-grained sense — a single operation within a capability that a given provider may or may not support. This is the one collision `Capability` introduces, and reserving the word closes it before it can drift. Whether `Feature` needs any runtime mechanism is unresolved; with one provider per capability in v1 it may never need expressing.
- **Capabilities are user-facing.** They are top-level command groups typed by humans and agents: `mc vcs …`, `mc ci …`, `mc runtime …`.
- **CI/CD maps to the token `ci`.** A slash cannot be a shell token. The term/token mismatch is deliberate and recorded in `CONTEXT.md` so it does not get "corrected" later. `cicd` is a mouthful nobody types; `pipeline` is the wrong noun, since a pipeline is an object *inside* the capability.
- **Go's stdlib has a `runtime` package.** Any file importing both needs an import alias. Accepted; mostly confined to provider internals.

Full reasoning and the rejected routes for image resolution: [issue #4](https://github.com/fwiedmann/misson-control/issues/4).
