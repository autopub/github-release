# githubrelease

[![Build Status](https://img.shields.io/travis/j0057/github-release.svg?maxAge=2592000)](https://travis-ci.org/j0057/github-release)

This project aims at streamlining the distribution of
[releases](https://github.com/blog/1547-release-your-software) on Github.

> I made it because it sucks to have to download a file from a server,
> only to upload it to Github from the desktop.
>
> It also sucks to download a file from github to your desktop, and then
> SCP it to a server.
>
> This thing works nicely from an SSH session.
>
> -- <cite>@j0057 on Wednesday, August 13, 2014</cite>


# examples

from the command-line:

```bash
# create a prerelease
$ githubrelease release jcfr/sandbox create --prerelease 1.0.0

# upload assets
$ githubrelease asset jcfr/sandbox 1.0.0 "dist/*"

# publish the release
$ githubrelease release jcfr/sandbox --publish 1.0.0

# or all together: create with custom name, upload assets, and publish
$ githubrelease release jcfr/sandbox create 2.0.0 --publish --name "Awesome 2.0" "dist/*"
```

... or even from python:

```python
from github_release import gh_release_create
gh_release_create("jcfr/sandbox ", "2.0.0", publish=True, name="Awesome 2.0", asset_pattern="dist/*")
```

*That said, if you are looking for a full fledged GitHub API support for
Python, you should probably look into project like [github3py](http://github3py.readthedocs.io/en/latest/) or
[PyGithub](https://github.com/PyGithub/PyGithub)*

# Table of Contents

   * [githubrelease](#githubrelease)
   * [examples](#examples)
   * [features](#features)
   * [installing](#installing)
   * [configuring](#configuring)
   * [using the cli](#using-the-cli)
      * [release command](#release-command)
      * [asset command](#asset-command)
      * [ref command](#ref-command)
   * [using the module](#using-the-module)
   * [testing](#testing)
   * [maintainers: how to make a release ?](#maintainers-how-to-make-a-release-)
   * [license](#license)

<!--
*Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc)*
-->

# features

* create release, pre-release, or draft release
* update any release metadata including referenced commit
* support wildcard expression (or list of wildcard expressions):
  * for upload or download of assets
  * for selectively deleting assets
* report download and upload progress
* allow deleting individual asset from a release
* authentication through `GITHUB_TOKEN` environment variable or `~/.netrc` file
* pure python, only depends on [requests](http://docs.python-requests.org/en/master/) and [click](http://pocco-click.readthedocs.io)


# installing

```bash
pip install githubrelease
```

# configuring

First, [generate a new token](https://help.github.com/articles/creating-an-access-token-for-command-line-use). It
should have at least the repo scope.

Then, there are two options:

* Set the `GITHUB_TOKEN` environment variable:

```bash
export GITHUB_TOKEN=YOUR_TOKEN
/path/to/command
```


* Put the key in `~/.netrc`, which should have mode 0600 (`-rw-------`):

```
machine api.github.com
login YOUR_TOKEN
password x-oauth-basic

machine uploads.github.com
login YOUR_TOKEN
password x-oauth-basic
```

where ``YOUR_TOKEN`` should be replaced with the generated token.

# using the cli

The package installs one CLI named ``githubrelease``.

```bash
$ githubrelease 
Usage: githubrelease [OPTIONS] COMMAND [ARGS]...

  A CLI to easily manage GitHub releases, assets and references.

Options:
  --progress / --no-progress  Display progress bar (default: yes).
  --help                      Show this message and exit.

Commands:
  asset    Manage release assets (upload, download, ...)...
  ref      Manage references (list, create, delete, ...)...
  release  Manage releases (list, create, delete, ...)...

Run 'githubrelease COMMAND --help' for more information on a command.
```

*For backward compatibility, it also installs `github-release` and `github-asset`*

## ``release`` command

This command deals with releases. The general usage is:

```bash
githubrelease release username/reponame command [tag] [options]
```

It understands the following commands:

| command       | parameters        | description                       |
|---------------|-------------------|-----------------------------------|
| list          |                   | list all releases                 |
| info          | tagname           | list one release                  |
| create        | tagname [options] | create a release                  |
| edit          | tagname [options] | Edit a release                    |
| delete        | tagname                | delete a release             |
| publish       | tagname [--prerelease] | make release public          |
| unpublish     | tagname [--prerelease] | make release draft           |
| release-notes | tagname           | use $EDITOR to edit release notes |

**Optional parameters:**

* create:

```bash
  --name NAME
  --publish
  --prerelease
  --target-commitish TARGET_COMMITISH
  [ASSET_PATTERN]...
```

* edit:

```bash
  --tag-name TAG_NAME
  --target-commitish TARGET_COMMITISH
  --name NAME
  --body BODY
  --draft
  --prerelease
  --dry-run
  --verbose
```


## ``asset`` command

This command deals with release assets. The general usage is:

```bash
githubrelease asset username/reponame command [tag] [filename] [options]
```

It understands the following commands:

| command   | parameters                 | description                                               |
|-----------|----------------------------|-----------------------------------------------------------|
| upload    | tagname filename           | upload a file to a release                                |
| download  |                            | download all files from all releases to current directory |
| download  | tagname                    | download all files from a release to current directory    |
| download  | tagname filename           | download file to current directory                        |
| delete    | tagname filename [options] | delete a file from a release                              |


**Optional parameters:**

* delete:

```bash
--keep-pattern KEEP_PATTERN
```


**Remarks:**

When specifying filenames, shell-like wildcards are supported, but make sure to
quote using single quotes, i.e. don't let the shell expand the wildcard pattern.

For the `download` command, you also need to specify a tagname of `'*'`


**Examples:**

```bash
# upload all example-project-1.4* files in /home/me/pkg
$ githubrelease asset octocat/example-project upload 1.4 '/home/me/pkg/example-project-1.4*'

# download all wheels from all releases
$ githubrelease asset octocat/example-project download '*' '*.whl'

# download all files from release 1.4
$ githubrelease asset octocat/example-project download 1.4

# download all files from example-project
$ githubrelease asset octocat/example-project download
```

## ``ref`` command

This command deals with git references. The general usage is:

```bash
githubrelease ref username/reponame command [options]
```

It understands the following commands:

| command   | parameters                             | description                                |
|-----------|----------------------------------------|--------------------------------------------|
| create    | ref sha                                | create reference (e.g heads/foo, tags/foo) |
| list      | [--tags] [--pattern PATTERN]           | list all references                        |
| delete    | pattern [--tags] [--keep-pattern KEEP_PATTERN] | delete selected references                 |


# using the module

The python API mirrors the command-line interface. Most of the available
function names follows this pattern:

```
gh_<COMMAND>_<COMMAND>
```

where the first ``<COMMAND>`` is either ``release``,
``asset`` or ``ref`` and the second one is any command respectively
documented above.

The parameters accepted by each function also mirrors the command-line
interface. The usual signature is:

```python
gh_<COMMAND>_<COMMAND>(repo_name, [param, [param,]] [option, [option]])
```

For example, here is the signature for ``gh_release_create``:

```python
def gh_release_create(repo_name, tag_name, 
                      name=None, publish=False, prerelease=False, target_commitish=None):
```

The type of each parameters or options can usually be inferred from
its name. If not, consider looking at [github_release.py](https://github.com/j0057/github-release/blob/update-readme/github_release.py).

```
repo_name        -> str
tag_name         -> str
name             -> str
publish          -> bool
prerelease       -> bool
target_commitish -> str
```

# testing

There are tests running automatically on TravisCI:
* coding style checks
* integration tests

Since the integration tests are expecting ``GITHUB_TOKEN`` to be set, they will
**NOT** be executed when pull request from fork are submitted. Indeed, setting
``GITHUB_TOKEN`` is required by the tests to reset and update [github-release-bot/github-release-test-py2](https://github.com/github-release-bot/github-release-test-py2)
and [github-release-bot/github-release-test-py3](https://github.com/github-release-bot/github-release-test-py3).

To execute the integration tests locally, and make sure your awesome contribution
is working as expected, you will have to:
* create a test repository with at least one commit (e.g `yourname/github-release-test`)
* set environment variable ``INTEGRATION_TEST_REPO_NAME=yourname/github-release-test``
* execute ``python setup.py test``

To execute a specific test, the following also works:

```bash
export GITHUB_TOKEN=YOUR_TOKEN
export INTEGRATION_TEST_REPO_NAME=yourname/github-release-test
$ pytest tests/test_integration_release_create.py::test_create_release
```

Moving forward, the plan would be to leverage tools like [betamax](http://betamax.readthedocs.io)
allowing to intercept every request made and attempting to find a matching request
that has already been intercepted and recorded.


# maintainers: how to make a release ?

1. Configure `~/.pypirc` as described [here](https://packaging.python.org/distributing/#uploading-your-project-to-pypi).

2. Make sure the cli and module work as expected

3. Tag the release. Requires a GPG key with signatures. For version *X.Y.Z*::

    ```bash
    git tag -s -m "githubrelease X.Y.Z" X.Y.Z origin/master
    ```

4. Create the source tarball and binary wheels::

    ```bash
    rm -rf dist/
    python setup.py sdist bdist_wheel
    ```

5. Upload the packages to the testing PyPI instance::

    ```bash
    twine upload --sign -r pypitest dist/*
    ```

6. Check the [PyPI testing package page](https://testpypi.python.org/pypi/githubrelease/).

7. Upload the packages to the PyPI instance::

    ```bash
    twine upload --sign dist/*
    ```

8. Check the [PyPI package page](https://pypi.python.org/pypi/githubrelease/).

9. Make sure the package can be installed::

    ```bash
    mkvirtualenv test-pip-install
    pip install githubrelease
    rmvirtualenv test-pip-install
    ```


# license

Written by Joost Molenaar ([@j0057](https://github.com/j0057)) and Jean-Christophe Fillion-Robin ([@jcfr](https://github.com/jcfr))

It is covered by the Apache License, Version 2.0:

http://www.apache.org/licenses/LICENSE-2.0

The license file was added at revision 0393859 on 2017-02-12, but you may
consider that the license applies to all prior revisions as well.