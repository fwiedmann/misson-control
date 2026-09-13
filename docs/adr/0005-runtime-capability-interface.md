---
status: accepted
---

# Runtime uses atomic service snapshots with instance drill-down

A Runtime provider instance is bound to one Stage and shares that Stage's Kubernetes client and informer factory. Each operation takes a provider-owned target for one configured Service. Issue #15 owns how provider config becomes that target.

The v1 interface has explicit typed operations equivalent to:

```text
Snapshot(ctx, target) -> RuntimeSnapshot
Observe(ctx, target) -> Observation[RuntimeSnapshot]
GetInstance(ctx, target, instanceRef) -> RuntimeInstanceDetail
ReadLogs(ctx, target, logQuery) -> LogRead
FollowLogs(ctx, target, logQuery) -> LogStream
Restart(ctx, target) -> error
```

Start and Stop are not in v1. Restart applies to the whole configured service target, returns after Kubernetes accepts the change, and has no result value. It patches the workload's Pod template. A paused Deployment and a StatefulSet using the `OnDelete` update strategy reject Restart with a typed unsupported-state error because a template change cannot start a rollout. Mission Control does not delete StatefulSet Pods to imitate one. Issue #10 owns Restart's dry-run, confirmation, and idempotency rules.

## Targets and snapshots

The Kubernetes provider supports Deployments and StatefulSets. Config must spell out `kind`, `workload`, and the app `container`; it never probes resource kinds or guesses which sidecar represents the Service. A missing workload is a successful snapshot with status `absent`. A missing configured app container is a terminal configuration error.

A snapshot is atomic. It contains:

- one operational status;
- desired, current, ready, available, and updated instance counts;
- the distinct declared-image and actual-image pairs observed for the app container, with instance counts;
- fixed-shape Runtime Instance summaries;
- provider-neutral problems.

The operational statuses are `absent`, `stopped`, `progressing`, `healthy`, `degraded`, `unavailable`, and `unknown`. After the absent check, desired zero means stopped. Healthy means every desired instance is updated and ready. Progressing means the controller reports valid forward progress. Once progress stalls, some availability means degraded and no availability means unavailable. Unknown is reserved for complete but inconsistent state. Observation lifecycle still reports data freshness separately.

A rollout may contain old and new images at once. Runtime preserves that set rather than claiming the desired image is already deployed. Each Container carries both its declared image reference and nullable runtime image identity. Issue #11 owns matching those values to a VCS Release.

One Kubernetes Pod lifetime maps to one Runtime Instance. Its opaque ref identifies that lifetime rather than only the reusable Pod name. Snapshot summaries stay small. `GetInstance` adds location and addresses, normalized conditions, fixed-shape Container details, and a bounded newest-first list of warning events. Raw Kubernetes objects do not cross the capability boundary. If the provider cannot read all state needed for a snapshot, a one-shot call fails and an Observation becomes degraded while retaining its last complete snapshot.

## Logs

Both log operations address exactly one Runtime Instance and one explicit Container. They only read the current container invocation. The provider does not merge pods or silently fall back to another container.

`ReadLogs` accepts a tail count or a since time. Core supplies a default tail of 100 lines. CLI output uses a small TOON metadata header followed by the exact log text. If presentation limits cut that fetched result, Core writes the full fetched text to a temporary file and reports its path and byte count.

`FollowLogs` is a lossless, back-pressured stream of fixed-shape records containing timestamp, Runtime Instance, Container, and message. Without an explicit tail or since bound it starts with new records only. The CLI renders those records as NDJSON. Normal container termination closes the stream cleanly. A transport loss returns an error rather than reconnecting with possible gaps or duplicates, and the stream never switches to a replacement Runtime Instance.

Full reasoning: [issue #9](https://github.com/fwiedmann/misson-control/issues/9).
