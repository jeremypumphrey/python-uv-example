# Python / uv Tool

## Overview
The UV is known to have these features: version locking, SHA-based package integrity, Minimum package age constraints
It helps reduce supply-chain risk (e.g., registry compromises like the recent Shai-Hulud incident)...

## Install
`brew install uv`: to install on macOS. 

`pip instal uv`: to install using python/pip 

## Initialization 
`uv init <project-name>`: create project and a file named pyproject.toml You can also use `uv init` in current folder to
do the same. 

Sample pyproject.toml file. 
```text
[project]
name = "my-uv-project"
version = "0.1.0"
description = "Sample project using uv for dependency management"
authors = ["Your Name <you@example.com>"]
requires-python = ">=3.14,<3.15"

[tool.uv]
# Python version for the project environment
python = "3.14"

[tool.uv.dependencies]
# Packages with version constraints
numpy = "~=2.4"           # Compatible with 2.4.*, <3.0
requests = "^3.0"         # Compatible with 3.0.*, <4.0
pandas = ">=2.0,<2.5"     # At least 2.0, but less than 2.5

[tool.uv.dev-dependencies]
# Dev-only packages
pytest = "^8.0"           # Latest 8.x for testing
black = "~=24.0"          # Latest 24.x for formatting

[build-system]
requires = ["uv>=1.0"]    # uv itself is required to build/manage this project
build-backend = "uv.build"
```

## Manage Python Version
`uv python install 3.14`: create a virtual environment for this project using Python 3.14.

`uv python pin 3.14`: locks the environment to exactly Python 3.14. 

`uv python upgrade 3.14`: upgrade the project environment’s Python to the latest 3.14.x version available.

##  Virtual Environment 
UV will create ./venv, the environment folder as needed, for example, the first time you issue command `uv add`. 

However, UV never activates a virtual environment in your shell, only running command `source .venv/bin/activate` will activate the 
virtual environment. 

The only times you need to activate a virtual environment is when you use `pip` 
instead of `uv pip`, use `python` instead of `uv run python`. 


## Install Packages

### Install (using pyproject.toml)
`uv install`: install all packages from tool.uv.dependencies under pyproject.toml to <project-root>/.venv/. 

`uv install --dev`: install all packages from tool.uv.dev-dependencies under pyproject.toml. Yes, you can manually 
create or edit tool.uv.dev-dependencies section. 

### Install (using uv add)
`uv add <package>`:  to install package to <project-root>/.venv/.

`uv add numpy~=2.4`: install the latest version of numpy that is compatible with 2.4, i.e., >=2.4.0 and <3.0.0. 

### Install (using uv pip install)
`uv pip install`: to use pip install packages(s) to where python/pip is located.  Be aware, you do not need to run 
`uv init` to use this command. Below is a full command example. 

```
uv pip install -r requirements.txt --target ./dependencies --python "$(which python)"  --exclude-newer "$(date -u -v-7d +%Y-%m-%d)"
```

`--python`  specifics the python interpreter location, it could be from a virtual environment. If you omit this option, 
you will need to be under ./venv environment, created by uv in some forms. 

`--exclude-newer` specifics to avoid packages got updated in the last 7 days. 


## Manage Package Versions

### uv.lock file 
Upon the following commands, `uv.lock` file is created. 

```text
uv install             # Install all dependencies from pyproject.toml
uv add numpy~=2.4      # Adds a new package, updates lockfile
uv python install      # Installing Python may trigger lockfile creation
uv lock                # Explicitly locks versions
```

Sample uv.lock file 
```text
# uv.lock

[metadata]
generated = "2026-01-13T12:00:00Z"
uv-version = "1.5.0"

[python]
version = "3.14.2"
hash = "sha256:abc123def456..."  # Optional, ensures exact interpreter

[packages]
numpy = { version = "2.5.3", hash = "sha256:xyz789..." }
requests = { version = "3.0.1", hash = "sha256:lmn456..." }
pandas = { version = "2.4.1", hash = "sha256:opq987..." }

[dev-packages]
pytest = { version = "8.0.2", hash = "sha256:rty321..." }
black = { version = "24.0.1", hash = "sha256:uvw654..." }

[constraints]
# These are your .toml constraints, recorded for reference
numpy = "~=2.4"
requests = "^3.0"
pandas = ">=2.0,<2.5"
pytest = "^8.0"
black = "~=24.0"

```

### uv lock command 
`uv lock --upgrade --exclude-newer "5 days"`: generates or updates the lockfile, resolves all dependencies and Python 
version according to your pyproject.toml constraints, stores the exact versions to ensure reproducibility across 
machines and CI. 

`--exclude-newer "5 days"`: use packages published at least 5 days ago. 

`--upgrade tells uv`:  to look for newer versions of all dependencies allowed by your constraints in pyproject.toml and 
update the lockfile (uv.lock) with them.



### uv sync command 
`uv sync --frozen --reinstall`: synchronizes your project environment with the lockfile (uv.lock). 

`--frozen`: tell uv not to change the lockfile. 

`--reinstall`:  forces uv to reinstall all packages in the environment. 

## Audit 
`uv pip audit --locked`: runs a security audit of your Python packages to check for known vulnerabilities in installed 
packages. Also, you do not need to activate virtual environment for this command. 


`--locked`: tells uv to audit exactly the versions listed in uv.lock, ignoring anything in pyproject.toml or the 
current venv. 

Example output. 
```text
Auditing locked dependencies...

Package    Version    Vulnerability ID      Severity
----------------------------------------------------
requests   3.0.1      PYSEC-2025-001      HIGH
numpy      2.5.3      None                 OK
pandas     2.4.1      None                 OK

Audit complete: 1 vulnerability found
```

## Run Script
`uv run python example.py`: this will run the python script example.py 


## CI/CD
Use uv in CI/CD pipelines (GitHub Actions example):
https://docs.astral.sh/uv/guides/integration/github/#using-uv-in-github-actions

# Repo Upgrade Process
The basic ideas is to install `uv`, and then replace `pip` with 'uv pip' using the --python option.  See below for examples. Full example is at https://github.com/BIAD/nci-aml-lambdas/blob/master/.github/workflows/main.yml. 

```
pip install uv
uv pip install -r requirements.txt --python "$(which python)"  --exclude-newer "$(date -u -d '4 days ago' +%Y-%m-%d)"
uv pip freeze --python "$(which python)"
uv pip install -r requirements-unit-test.txt --python "$(which python)"
```
