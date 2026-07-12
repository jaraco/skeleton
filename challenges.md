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

Github has some nice features to link mentions in commits to issues and pull requests, including taking actions such as closing issues. When applying commits from the skeleton that mention fixing a common concern can [unintentionally affect downstream projects](https://github.com/jaraco/skeleton/issues/87) unless the committer is careful to use project-absolute references (e.g. "jaraco/skeleton#27" vs. "#27").
