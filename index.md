# Overview

While maintaining dozens of projects in PyPI, jaraco derives best practices for project distribution and publishes them in the [skeleton repo](https://github.com/jaraco/skeleton), a Git repo capturing the evolution and culmination of these best practices. The repo contains the scaffolding of a Python project jaraco [introduced in this blog](https://blog.jaraco.com/a-project-skeleton-for-python-projects/). The skeleton provides a system for Python projects to re-use techniques and inherit advances when maintaining and publishing.

Skeleton lives upstream of its downstream consumers; see [SCM-Managed Approach](scm-approach.md) for more background on the interaction between repositories.

See [design](design.md) and [challenges](challenges.md) for more details.

## Alternatives

An alternative to maintaining this boilerplate in a shared repo is not to have the boilerplate at all, a concept implemented by the [Coherent System](/coherent-oss/system).

# Contributing

This section provides guidance on contributing to skeleton-based projects. If contributing to the skeleton itself, see the [separate guidance](skeleton-contrib.md).

## Running Checks

The skeleton relies on [tox](https://pypi.org/project/tox) to drive development tasks. See [design](design.md) for the rationale behind the tools referenced below.

Run all checks (tests, formatting, linting, type checks, and coverage) on the default Python with:

```
tox
```

To run under a specific Python:

```
tox -e py314
```

List the other available environments and what each does with:

```
tox list
```

Build the documentation with:

```
tox -e docs
```

Disable an individual check for a run by passing `-p no:{plugin}` through to pytest. For example, to skip coverage:

```
tox -- -p no:cov
```

The checks only report violations; they never modify your code. Apply formatting and lint fixes with:

```
ruff format .
ruff check --fix .
```

## Adding release notes

Record every user-facing change as a [Towncrier](https://towncrier.readthedocs.io/) news fragment under `newsfragments/`; release notes are assembled from these fragments. Name the file using the issue ID (or pull request ID if no relevant issue) and scope (removal, feature, bugfix). If no issue ID is available, use some unique identifier prefixed by a `+` (i.e. `+1dbf102a.feature.rst`).

## Releasing

When main is stable, cut a release from main using `tox -e finalize`, which will assemble and commit the release notes and tag the commit. Push that commit to release to PyPI using GitHub Actions.
