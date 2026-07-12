# Contributing to the Skeleton

This guidance applies to changes to the skeleton project *itself*. Because the skeleton's history is merged into each of its downstream consumers, contributions here carry extra constraints.

## Validation

There are no checks or tests. Consider validating any changes with some downstream project first and then porting (or cherry-picking) the commit to skeleton.

## Squash merge pull requests

Always squash merge pull requests. Every commit on the skeleton's main branch is inherited by every downstream project, so collapsing each change to a single commit keeps that shared history as small and legible as possible.

## Use repo-absolute references

Always use repo-absolute references in commit messages and news fragments — for example `jaraco/skeleton#123`, never a bare `#123`. Because these commits are merged into downstream projects, a bare reference would resolve against the wrong repository and notify unrelated issues. See [Commit Integrations Mismatch](challenges.md) for rationale.
