# Challenges

Because this approach applies concerns across repos, it does violate some assumptions leading to undesirable outcomes.


## History is Forever

The history accumulates and each project that adopts it gets the full history. Even a brand new project will get commits going back as far as the main history goes. As a result, cruft accumulates and multiplies across projects.

The [Periodic Collapse](scm-approach.md) attempts to alleviate this pressure by occasionally (rarely) collapsing the history, but this approach adds its own downsides:

- the true history is obscured
- existing histories are broken until the handoff commit is pulled
- attribution is lost


## Continuous Integration Mismatch

Because CI instructions for "best practices" are stored in the repo, it's not possible to supply CI instructions for both:

- downstream projects, and
- the skeleton itself.

Best case, the tests in the skeleton repo are degenerate and pass. Worst case, the tests fail because they rely on factors expected to be supplied downstream (e.g. doc builds currently fail because they rely on the root package name being supplied to be documented).

Moreover, it's not viable to test aspects specific to the skeleton that should not appear/apply downstream, such as to check that commit messages in the skeleton meet certain criteria.


## Commit Integrations Mismatch

Github has some nice features to link mentions in commits to issues and pull requests, including taking actions such as closing issues. Applying commits from the skeleton that mention fixing a common concern can [unintentionally affect downstream projects](https://github.com/jaraco/skeleton/issues/87) unless the committer is careful to use repo-qualified references (e.g. "jaraco/skeleton#27" vs. "#27").


## Version Pinning and Skew

The skeleton's guiding philosophy is to carry as little debt as possible, and pinning tool versions is treated as debt. Wherever a tool allows it, the skeleton leaves versions unpinned so that downstream projects inherit upstream advances (and bug fixes) automatically, without a coordinated bump across dozens of repos — see, for example, the deliberately [unpinned GitHub Actions](https://github.com/pypa/setuptools/issues/4025).

Some tools, however, *require* a pinned version to function. [pre-commit](https://pre-commit.com) is the notable case: its `.pre-commit-config.yaml` must name an explicit `rev` for each hook repo (e.g. `ruff-pre-commit`). That pin is a standing source of debt and toil. It inevitably drifts out of sync with the *unpinned* `ruff` that the test extra resolves in CI, so the two disagree: `pre-commit` autofixes against one (older) rule set while `pytest-ruff` checks against a newer one — and a newer `ruff --fix` can even strip the `# noqa` and `import X as X` re-export markers the code depends on. This recurs with each `ruff` release that expands its default rule selection (see [jaraco/skeleton#180](https://github.com/jaraco/skeleton/issues/180) and [jaraco/skeleton#210](https://github.com/jaraco/skeleton/issues/210)).

For this reason the skeleton treats `.pre-commit-config.yaml` as **best-effort, non-authoritative, and contributor-maintained**. It is not exercised by the skeleton's own checks, is expected to be stale at any given time, and is provided as a convenience only ([use at your own risk](https://github.com/jaraco/skeleton/issues/109#issuecomment-1932575587)). The authoritative lint and format checks are the ones driven by tox/pytest; when the pre-commit config and those checks disagree, the pre-commit config is the one that's wrong.
