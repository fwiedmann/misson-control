---
status: accepted
---

# CI/CD is a typed capability around pipelines, manual actions, and release

The CI/CD capability is provider-neutral Mission Control vocabulary, not a thin GitLab API wrapper. A provider instance is bound to one Service and exposes explicit typed operations for pipelines, pipeline observation, manual actions, running pipelines, releasing a service, and pipeline-candidate lookup for image-to-source resolution; Core addresses pipelines by opaque refs and normalizes provider statuses into a small enum.

A CI/CD Observation targets a whole PipelineSnapshot: pipeline summary, ordered pipeline phases, jobs, and manual actions. GitLab CI "stages" are called **pipeline phases** so they do not collide with Mission Control **Stage**, and GitLab manual jobs are **manual actions** because Core cares that they can be triggered, not that GitLab models them as jobs.

`RunPipeline` takes an explicit source ref and no arbitrary variables in v1. `ReleaseService` is first-class: it takes a Mission Control Stage and explicit source ref, uses provider config's stage-keyed allowlist of release job names, acts on the newest pipeline for that ref with at least one playable configured release action, returns not-ready if the newest pipeline is still running, triggers every matching action in deterministic pipeline order, and reports per-action outcomes with overall success only when every action triggers.

CI/CD owns candidate lookup for image-to-source resolution, but not the final matching policy. Given an image-ish version string, it returns plausible pipeline candidates; issue #11 decides how those candidates are matched to Runtime state. List and detail operations remain atomic, matching the VCS capability. Observation degradation and per-action release outcomes are the explicit exceptions.

Full reasoning: [issue #8](https://github.com/fwiedmann/misson-control/issues/8).
