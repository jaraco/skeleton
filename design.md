# Design

The features/techniques employed by the skeleton include:

- Standards-based build relying on Setuptools as the build tool
- Setuptools declarative configuration using pyproject.toml
- tox for running tests
- A README.rst as reStructuredText with some popular badges, but with Read the Docs badges commented out
- A NEWS.rst file intended for publishing release notes about the project, constructed from Towncrier RST fragments
- Use of [Ruff](https://github.com/astral-sh/ruff) for linting and formatting
- Integrated type checking through [mypy](https://github.com/python/mypy/)
- Use of [diff-cover](https://pypi.org/project/diff-cover) to encourage test coverage

## Packaging Conventions

A pyproject.toml is included and implements the following features:

- Enable PEP 517 and PEP 518 to build the project on Setuptools.
- Reads the README.rst file into the long description
- Some common Trove classifiers
- Includes all packages discovered in the repo
- Data files in the package are also included (not just Python files)
- Declares the required Python versions
- Declares install requirements (empty by default)
- Supplies two sets of "optional dependencies":
  - test: requirements for running tests
  - doc: requirements for building docs
  - these dependencies (sometimes called "extras") split the declaration into "upstream" (requirements as declared by the skeleton) and "local" (those specific to the local project); these markers help avoid merge conflicts
- Placeholder for defining entry points

Additionally, the pyproject.toml file declares `[tool.setuptools_scm]`, which enables [setuptools_scm](https://pypi.org/project/setuptools_scm) to do two things:

- derive the project version from SCM tags
- ensure that all files committed to the repo are automatically included in releases


### Compatibility Modules

Projects relying on `skeleton` are advised to organise their own internal
polyfills, backports, workarounds and version conditional imports into a
series of separated modules under the `compat` subpackage.
These modules provide compatibility layers or shims and often ensure code runs
smoothly across different Python versions.

Examples of such practice include
[keyring.compat](https://github.com/jaraco/keyring/blob/main/keyring/compat/),
[zipp.compat](https://github.com/jaraco/zipp/blob/main/zipp/compat), and
[setuptools.compat](https://github.com/pypa/setuptools/tree/main/setuptools/compat).

In the case of shims and conditional imports that depend on specific versions
of the interpreter, the module name should reflect the Python version that requires the
legacy behavior. For example, the module `setuptools.compat.py312` supports
compatibility with Python 3.12 and earlier.
This naming convention is beneficial because it signals when the code
can be removed. When support for Python 3.12 is dropped
(i.e., when `requires-python = ">=3.13"` is added to `pyproject.toml`),
imports of the module `py312` will be easily identifiable as removable debt.

Please note that these modules are implementation details and -
*unless explicitly documented otherwise by each particular project* - not part of the public API.


## Running Tests

The skeleton assumes the developer has [tox](https://pypi.org/project/tox) installed. The developer is expected to run `tox` to run tests on the current Python version using [pytest](https://pypi.org/project/pytest).

The test suite is configured not only to run the project's tests as discovered by pytest, but also includes several plugins to perform other checks:

- `pytest-ruff` performs lint and formatting checks using [ruff](https://github.com/astral-sh/ruff)
- `pytest-mypy` performs type checks using [mypy](https://github.com/python/mypy)
- `pytest-checkdocs` ensures that the README renders without errors
- `pytest-cov` reports the coverage of the project

Many of these plugins are enabled through `pytest-enabler` to allow easy disablement of any one of them. Simply pass `-p no:{plugin}` (e.g. `-p no:cov`) to pytest (e.g. `tox -- -p no:cov`) to disable the plugin for that run.

Other environments (invoked with `tox -e {name}`) are available. Use `tox list` for a description of each.

A pytest.ini is included to define common options around running tests. In particular:

- rely on default test discovery in the current directory
- avoid recursing into common directories not containing tests
- run doctests on modules and invoke Ruff tests
- filters out known warnings caused by libraries/functionality included by the skeleton

These checks are non-invasive; they don't automatically modify the code, but merely report on violations. Correct the violations manually or consider running the relevant tools (e.g. `ruff format .` or `ruff check --fix .`). A pre-commit config exists, but due to [maintenance challenges](https://github.com/jaraco/skeleton/issues/109#issuecomment-1932575587) is unsupported (use at your own risk).

## Continuous Integration

The project is pre-configured to run Continuous Integration tests.

### Github Actions

[Github Actions](https://docs.github.com/en/free-pro-team@latest/actions) are the preferred provider as they provide free, fast, multi-platform services with straightforward configuration. Configured in `.github/workflows`.

Features include:
- test against multiple Python versions
- run on late (and updated) platform versions
- automated releases of tagged commits
- [automatic merging of PRs](https://github.com/marketplace/actions/merge-pull-requests) (requires [protecting branches with required status checks](https://docs.github.com/en/free-pro-team@latest/github/administering-a-repository/enabling-required-status-checks), [not possible through API](https://github.community/t/set-all-status-checks-to-be-required-as-branch-protection-using-the-github-api/119493))
- [unpinned actions](https://github.com/pypa/setuptools/issues/4025)
- [efficient resource consumption](https://blog.jaraco.com/efficient-use-of-ci-resources/)

### Continuous Deployments

In addition to running tests, an additional publish stage is configured to automatically release tagged commits to PyPI using [API tokens](https://pypi.org/help/#apitoken). The release process expects an authorized token to be configured with each Github project (or org) `PYPI_TOKEN` [secret](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets). Example:

```
pip-run jaraco.develop -- -m jaraco.develop.add-github-secrets
```

## Building Documentation

Documentation is automatically built by [Read the Docs](https://readthedocs.org) when the project is registered with it, by way of the .readthedocs.yml file. To test the docs build manually, a tox env may be invoked as `tox -e docs`. Both techniques rely on the dependencies declared in `pyproject.toml/project.optional-dependencies.doc`.

In addition to building the Sphinx docs scaffolded in `docs/`, the docs build a `history.html` file that first injects release dates and hyperlinks into the CHANGES.rst before incorporating it as history in the docs.

## Cutting releases

By default, tagged commits are released through the continuous integration deploy stage.

Releases may also be cut manually by invoking the tox environment `release` with the PyPI token set as the TWINE_PASSWORD:

```
TWINE_PASSWORD={token} tox -e release
```

## Ignoring Artifacts

This project does not include a `.gitignore` module or provides a very minimal one because this project holds the philosophy that it's preferable to specify ignores at the most relevant level, and so .gitignore for a project should specify elements unique to _that project_ and not elements peculiar to the language or system or user.

Instead, skeleton recommends users to implement ignores for the language(s), system, and user in the development environment. For example, jaraco has the following in `~/.gitconfig`:

```
[core]
  excludesfile = ~/.gitignore_global
```

Referencing this [`.gitignore_global`](https://github.com/jaraco/dotfiles/blob/main/.gitignore_global).

As you can see, this file contains all of the commonly encountered ignorables when developing node.js and Python projects using PyCharm or emacs or git on all the common platforms. This simple configuration, linked in each development environment, avoids the need to configure (and sync) each downstream project with the aggregate configuration of jaraco's environments and the environments of each of the contributors to each of the projects the contributiors may touch.

It's not a perfect alignment of concerns to projects, but it's a dramatically simpler approach saving hundreds of commits that can be readily adopted by any user and is strongly recommended for skeleton-based projects.

See [gitignore docs](https://git-scm.com/docs/gitignore) for more details regarding configuring ignore files. For users only incidentally involved with skeleton-derived projects, consider downloading jaraco's `.gitignore_global` to the checked-out project:

```shell
wget https://raw.githubusercontent.com/jaraco/dotfiles/main/.gitignore_global -O - >> .git/info/exclude
```

Note: See [dotfiles](https://docs.github.com/en/codespaces/setting-your-user-preferences/personalizing-github-codespaces-for-your-account#dotfiles) for guidance on applying these settings in GitHub Codespaces.
