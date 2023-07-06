# Overview

This project is merged with [skeleton](https://github.com/jaraco/skeleton). What is skeleton? It's the scaffolding of a Python project jaraco [introduced in his blog](https://blog.jaraco.com/a-project-skeleton-for-python-projects/). It seeks to provide a means to re-use techniques and inherit advances when managing projects for distribution.

## An SCM-Managed Approach

While maintaining dozens of projects in PyPI, jaraco derives best practices for project distribution and publishes them in the [skeleton repo](https://github.com/jaraco/skeleton), a Git repo capturing the evolution and culmination of these best practices.

It's intended to be used by a new or existing project to adopt these practices and honed and proven techniques. Adopters are encouraged to use the project directly and maintain a small deviation from the technique, make their own fork for more substantial changes unique to their environment or preferences, simply adopt the skeleton once and abandon it thereafter, or use it as a reference from which to cherry-pick ideas.

The primary advantage to using an SCM for maintaining these techniques is that those tools help facilitate the merge between the template and its adopting projects.

Another advantage to using an SCM-managed approach is that tools like GitHub recognize that a change in the skeleton is the _same change_ across all projects that merge with that skeleton. Without the ancestry, with a traditional copy/paste approach, a [commit like this](https://github.com/jaraco/skeleton/commit/12eed1326e1bc26ce256e7b3f8cd8d3a5beab2d5) would produce notifications in the upstream project issue for each and every application, but because it's centralized, GitHub provides just the one notification when the change (and its commit hash) is added to the skeleton.

# Usage

## new projects

To use skeleton for a new project, simply pull the skeleton into a new project:

```
$ git init my-new-project
$ cd my-new-project
$ git pull gh://jaraco/skeleton
```

Now customize the project to suit your individual project needs.

In particular, make some replacements:

- `PROJECT`: name of the project in PYPI
- `PROJECT_PATH`: org/name of the project in Github
- `PROJECT_DESCRIPTION`: a description of the project
- `PROJECT_RTD`: (optional) the RTD-mangled name of the project

## existing projects

If starting from an existing project, incorporate the skeleton by merging it into the codebase.

```
$ git merge skeleton --allow-unrelated-histories
```

The `--allow-unrelated-histories` is necessary on the first merge because the history from the skeleton was previously unrelated to the existing codebase. Resolve any merge conflicts and commit to the master, and thereafter the project is based on the shared skeleton.

## Updating

Whenever a change is needed or desired for the general technique for packaging, it can be made in the skeleton project and then merged into each of the derived projects as needed, recommended before each release. As a result, features and best practices for packaging are centrally maintained and readily trickle into a whole suite of packages. This technique lowers the amount of tedious work necessary to create or maintain a project, and coupled with other techniques like continuous integration and deployment, lowers the cost of creating and maintaining refined Python projects to just a few, familiar Git operations.

For example, here's a session of the [path project](https://pypi.org/project/path) pulling non-conflicting changes from the skeleton:

<img src="https://raw.githubusercontent.com/jaraco/skeleton/gh-pages/docs/refresh.svg">

Additionally, the author maintains a [routine to mechanically apply](https://github.com/jaraco/jaraco.develop/blob/main/jaraco/develop/update-projects.py) the changes from the skeleton or other "upstream" bases.

Thereafter, the target project can make whatever customizations it deems relevant to the scaffolding. The project may even at some point decide that the divergence is too great to merit renewed merging with the original skeleton. This approach applies maximal guidance while creating minimal constraints.

## Periodic Collapse

In late 2020, this project [introduced](https://github.com/jaraco/skeleton/issues/27) the idea of a periodic but infrequent (O(years)) collapse of commits to limit the number of commits a new consumer will need to accept to adopt the skeleton.

The full history of commits is collapsed into a single commit and that commit becomes the new mainline head.

When one of these collapse operations happens, any project that previously pulled from the skeleton will no longer have a related history with that new main branch. For those projects, the skeleton provides a "handoff" branch that reconciles the two branches. Any project that has previously merged with the skeleton but now gets an error "fatal: refusing to merge unrelated histories" should instead use the handoff branch once to incorporate the new main branch.

```
$ git pull https://github.com/jaraco/skeleton 2020-handoff
```

This handoff needs to be pulled just once and thereafter the project can pull from the main head.

Here's what the tree looks like following a handoff:

<img src="https://raw.githubusercontent.com/jaraco/skeleton/gh-pages/docs/handoff.png">

The archive and handoff branches from prior collapses are indicate here:

| refresh | archive         | handoff      |
|---------|-----------------|--------------|
| 2020-12 | archive/2020-12 | 2020-handoff |
| 2023-07 | archive/2023-07 | 2023-handoff |

# Features

The features/techniques employed by the skeleton include:

- PEP 517/518-based build relying on Setuptools as the build tool
- Setuptools declarative configuration using setup.cfg
- tox for running tests
- A README.rst as reStructuredText with some popular badges, but with Read the Docs badges commented out
- A CHANGES.rst file intended for publishing release notes about the project
- Use of [Black](https://black.readthedocs.io/en/stable/) for code formatting (disabled on unsupported Python 3.5 and earlier)
- Integrated type checking through [mypy](https://github.com/python/mypy/)
- Dependabot enabled to enable supply chain security

## Packaging Conventions

A pyproject.toml is included to enable PEP 517 and PEP 518 compatibility and declares the requirements necessary to build the project on Setuptools (a minimum version compatible with setup.cfg declarative config).

The setup.cfg file implements the following features:

- Assumes universal wheel for release
- Advertises the project's LICENSE file (MIT by default)
- Reads the README.rst file into the long description
- Some common Trove classifiers
- Includes all packages discovered in the repo
- Data files in the package are also included (not just Python files)
- Declares the required Python versions
- Declares install requirements (empty by default)
- Supplies two 'extras':
  - testing: requirements for running tests
  - docs: requirements for building docs
  - these extras split the declaration into "upstream" (requirements as declared by the skeleton) and "local" (those specific to the local project); these markers help avoid merge conflicts
- Placeholder for defining entry points

Additionally, the pyproject.toml file declares `[tool.setuptools_scm]`, which enables [setuptools_scm](https://pypi.org/project/setuptools_scm) to do two things:

- derive the project version from SCM tags
- ensure that all files committed to the repo are automatically included in releases

## Running Tests

The skeleton assumes the developer has [tox](https://pypi.org/project/tox) installed. The developer is expected to run `tox` to run tests on the current Python version using [pytest](https://pypi.org/project/pytest).

The test suite is configured not only to run the project's tests as discovered by pytest, but also includes several plugins to perform other checks:

- `pytest-ruff` performs lint checks using [ruff](https://github.com/astral-sh/ruff)
- `pytest-black` checks that the style conforms to [black](https://pypi.org/project/black/)
- `pytest-mypy` performs type checks using [mypy](https://github.com/python/mypy)
- `pytest-checkdocs` ensures that the README renders without errors
- `pytest-cov` reports the coverage of the project

Many of these plugins are enabled through `pytest-enabler` to allow easy disablement of any one of them. Simply pass `-p no:{plugin}` (e.g. `-p no:cov`) to pytest (e.g. `tox -- -p no:cov`) to disable the plugin for that run.

Other environments (invoked with `tox -e {name}`) supplied include:

  - a `docs` environment to build the documentation
  - a `release` environment to publish the package to PyPI

A pytest.ini is included to define common options around running tests. In particular:

- rely on default test discovery in the current directory
- avoid recursing into common directories not containing tests
- run doctests on modules and invoke Ruff tests
- filters out known warnings caused by libraries/functionality included by the skeleton

## Continuous Integration

The project is pre-configured to run Continuous Integration tests.

### Github Actions

[Github Actions](https://docs.github.com/en/free-pro-team@latest/actions) are the preferred provider as they provide free, fast, multi-platform services with straightforward configuration. Configured in `.github/workflows`.

Features include:
- test against multiple Python versions
- run on late (and updated) platform versions
- automated releases of tagged commits
- [automatic merging of PRs](https://github.com/marketplace/actions/merge-pull-requests) (requires [protecting branches with required status checks](https://docs.github.com/en/free-pro-team@latest/github/administering-a-repository/enabling-required-status-checks), [not possible through API](https://github.community/t/set-all-status-checks-to-be-required-as-branch-protection-using-the-github-api/119493))


### Continuous Deployments

In addition to running tests, an additional publish stage is configured to automatically release tagged commits to PyPI using [API tokens](https://pypi.org/help/#apitoken). The release process expects an authorized token to be configured with each Github project (or org) `PYPI_TOKEN` [secret](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets). Example:

```
pip-run jaraco.develop -- -m jaraco.develop.add-github-secrets
```

## Building Documentation

Documentation is automatically built by [Read the Docs](https://readthedocs.org) when the project is registered with it, by way of the .readthedocs.yml file. To test the docs build manually, a tox env may be invoked as `tox -e docs`. Both techniques rely on the dependencies declared in `setup.cfg/options.extras_require.docs`.

In addition to building the Sphinx docs scaffolded in `docs/`, the docs build a `history.html` file that first injects release dates and hyperlinks into the CHANGES.rst before incorporating it as history in the docs.

## Cutting releases

By default, tagged commits are released through the continuous integration deploy stage.

Releases may also be cut manually by invoking the tox environment `release` with the PyPI token set as the TWINE_PASSWORD:

```
TWINE_PASSWORD={token} tox -e release
```

## Ignoring Artifacts

This project does not include a `.gitignore` module because this project holds the philosophy that it's preferable to specify ignores at the most relevant level, and so .gitignore for a project should specify elements unique to _that project_ and not elements peculiar to the language or system or user.

Instead, skeleton recommends users to implement ignores for the language(s), system, and user in the development environment. For example, jaraco has the following in `~/.gitconfig`:

```
[core]
  excludesfile = /Users/jaraco/.gitignore_global
```

And in the `.gitignore_global`, something like the following:

```
*~
.DS_Store
.cache
.eggs
__pycache__
build
dist
*.egg-info
.tox
.xprocess
*.orig
*.pyc
node_modules
.mypy_cache
.pytest_cache
.env
.coverage
pip-wheel-metadata
.coverage.*
.idea
coverage.xml
```

As you can see, this file contains all of the commonly encountered ignorables when developing node.js and Python projects using PyCharm or emacs or git on a Mac. This simple configuration, linked in each development environment, avoids the need to configure (and sync) each downstream project with the aggregate configuration of jaraco's environments and the environments of each of the contributors to each of the projects the contributiors may touch.

It's not a perfect alignment of concerns to projects, but it's a dramatically simpler approach saving hundreds of commits that can be readily adopted by any user and is recommended for skeleton-based projects.
