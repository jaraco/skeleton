# Overview

While maintaining dozens of projects in PyPI, jaraco derives best practices for project distribution and publishes them in the [skeleton repo](https://github.com/jaraco/skeleton), a Git repo capturing the evolution and culmination of these best practices. The repo contains the scaffolding of a Python project jaraco [introduced in this blog](https://blog.jaraco.com/a-project-skeleton-for-python-projects/). The skeleton provides a system for Python projects to re-use techniques and inherit advances when maintaining and publishing.

Skeleton lives upstream of its downstream consumers; see [SCM-Managed Approach](scm-approach.md) for more background on the interaction between repositories.

See [design](design.md) and [challenges](challenges.md) for more details.

## Alternatives

An alternative to maintaining this boilerplate in a shared repo is not to have the boilerplate at all, a concept implemented by the [Coherent System](/coherent-oss/system).
