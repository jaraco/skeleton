# Contributing to the Skeleton

This guidance applies to changes to the skeleton project *itself*. Because the skeleton's history is merged into each of its downstream consumers, contributions here carry extra constraints.

The source for this page lives on the `gh-pages` branch of `jaraco/skeleton` (as `skeleton-contrib.md`); the rendered form is published at <https://blog.jaraco.com/skeleton>.

## Validation

The skeleton carries no tests or checks of its own. More broadly, most of the machinery it provides — tox, towncrier, the CI checks — does not apply to the skeleton itself; those features exist to serve downstream projects. To validate a change, apply it to a real downstream project first, then port (or cherry-pick) the commit back to the skeleton.

## History lives in commit messages

The skeleton keeps no changelog of its own, so its history is conveyed entirely through its commit messages. Treat each message as the durable, user-facing record of what changed and why.

## Squash merge pull requests

Always squash merge pull requests. Every commit on the skeleton's main branch is inherited by every downstream project, so collapsing each change to a single commit keeps that shared history as small and legible as possible.

## Use repo-qualified references

Always use repo-qualified references in commit messages and any tracked file (code comments, configuration, etc.) — for example `jaraco/skeleton#123`, never a bare `#123`. Because these commits are merged into downstream projects, a bare reference would resolve against the wrong repository and notify unrelated issues. See [Commit Integrations Mismatch](challenges.md#commit-integrations-mismatch) for rationale.

## Issues

The skeleton's issue tracker covers both defects in the skeleton implementation and systemic problems common to its downstream projects — even when the latter cannot be resolved in the skeleton itself.

## News fragments

Avoid news fragments in the skeleton. A fragment added here propagates to every downstream project on merge, and the skeleton can only remove it once all of them have rendered and deleted their copies — remove it any sooner and late-syncing projects never receive it. That is an unbounded coordination burden.

If a fragment is truly unavoidable, it must be anonymous (a `+` filename). A numbered fragment makes towncrier emit a bare `#N` that resolves against each downstream repository rather than the skeleton, and — per the rule above — a fragment cannot carry a repo-qualified reference.
