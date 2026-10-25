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

Some tools, however, *require* a pinned version to function. [pre-commit](https://pre-commit.com) is the notable case: its `.pre-commit-config.yaml` must name an explicit `rev` for each remote hook repo. Historically, the skeleton ran `ruff` through the `ruff-pre-commit` repo, and that pin inevitably drifted out of sync with the *unpinned* `ruff` that the `check` extra resolves in CI, so the two disagreed: `pre-commit` autofixed against one (older) rule set while `pytest-ruff` checked against a newer one — and a newer `ruff --fix` could even strip the `# noqa` and `import X as X` re-export markers the code depends on (see [jaraco/skeleton#180](https://github.com/jaraco/skeleton/issues/180) and [jaraco/skeleton#210](https://github.com/jaraco/skeleton/issues/210)).

To eliminate that skew, the skeleton runs `ruff` as a [`repo: local`](https://pre-commit.com/#repository-local-hooks) hook ([jaraco/skeleton#204](https://github.com/jaraco/skeleton/pull/204)). A preceding local hook installs the project's `check` extra (using `uv pip` if available, else `pip`) into the active environment, and the `ruff-check` and `ruff-format` hooks then invoke that `ruff`. The version therefore comes from the project's own dependency declarations — the same source CI uses — with no duplicated pin. Note that, because these hooks use `language: system`, they install into and run from whatever Python environment is active when `pre-commit` runs.

The only remaining pin is for [`pre-commit-hooks`](https://github.com/pre-commit/pre-commit-hooks), which supplies a few language-agnostic autofixers (trailing whitespace, end-of-file, line endings, byte-order marker). These don't overlap with the project's linters, so the pin carries no risk of conflicting with the authoritative checks.

The authoritative lint and format checks remain the ones driven by tox/pytest; `pre-commit` is offered as a convenience for autofixing before commit. Because the hooks are local, the config isn't compatible with [pre-commit.ci](https://pre-commit.ci). Projects wanting autofixes on pull requests can instead use [pre-commit.ci lite](https://pre-commit.ci/lite) or [autofix.ci](https://autofix.ci/) (optionally with [prek](https://github.com/j178/prek) as the runner), which run the hooks in the project's own CI.
