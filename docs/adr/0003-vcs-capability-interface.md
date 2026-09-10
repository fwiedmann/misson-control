---
status: accepted
---

# Typed VCS queries with Core-owned checkout

The VCS Capability uses explicit, provider-neutral list and detail operations for Merge Requests, Releases, and branches. Core owns local checkout rather than making the GitLab Provider act on a working copy. This keeps the remote Provider interface type-safe and prevents local git state from leaking into a Provider that otherwise wraps a remote system.

## Provider interface

A VCS Provider instance is bound to one Service. It exposes typed operations equivalent to:

- list and get Merge Requests;
- list and get Releases;
- list and get branches;
- compare two branches;
- test whether a git remote URL identifies the configured repository.

The interface uses explicit methods rather than generic list/get request unions. The extra method names preserve compile-time request/result pairing and keep TUI and CLI callers free of type switches.

VCS has no Observation in v1. Callers request snapshots.

## Data contract

List operations return fixed-shape summaries. Single-item operations return details that add large text and collections. AXI field selection happens above the Provider interface.

All lists use opaque cursors and return a `Total` with separate `Known` and count values. A Provider never reports an unavailable total as zero. Lists are atomic and do not return partial pages.

Merge-request filters are typed and cover state, participation role, user, source and target branches, update time, and text search. Multiple roles combine with OR; other filters combine with AND. A role without an explicit user refers to the authenticated user. Results sort by most recent update.

Every repository tag appears as a Release. Provider release metadata enriches the same fixed shape with nullable name, release time, URL, and stored changelog. Tag-only entries leave those fields absent. Releases sort by release time when available and otherwise by the tagged commit's time.

Branch lists sort by the head commit's time, newest first. Ahead and behind counts come from an explicit branch comparison operation because GitLab needs two compare calls. Each count carries whether it is known; a compare timeout produces unknown, never a false zero.

## Checkout

Checkout remains a user-facing VCS action, but Core performs it against the current directory. Core reads the repository's remotes and asks the Provider which remote matches the configured Service. The Provider only normalizes and compares URLs; it never reads the working copy or invokes git.

After verifying a clean working tree and a matching remote, Core fetches the named branch and switches to an existing matching local branch or creates its tracking branch. It never pulls, stashes, resets, or forces. The separate mutation-safety decision governs dry-run, execution, and confirmation behavior.

Full reasoning: [Design the VCS capability interface](https://github.com/fwiedmann/misson-control/issues/7).
