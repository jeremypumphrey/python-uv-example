# Python / uv Tool

## Overview
Example Python project using uv with:
- Version locking
- SHA-based package integrity
- Minimum package age constraints

This setup helps reduce supply-chain risk (e.g., registry compromises like the recent Shai-Hulud incident).

## Install
Install uv (macOS, easiest option):

brew install uv

## Initialize Repository
1. Initialize a new project:

uv init repo-name

This creates a new project directory with a pyproject.toml.

## Python Version
2. Install and manage a specific Python version:

uv python install 3.14
uv python pin 3.14
uv python upgrade 3.14

- install downloads the version
- pin locks the project to that version
- upgrade updates to the latest supported patch release

## Dependencies
3. Declare broad dependencies in pyproject.toml, pinned only to major/minor versions using ~=.

These files should be human-editable, but the easiest approach is using uv add:

uv add numpy~=2.4

Equivalent specifier behavior:
- foo~=1.2 → foo>=1.2,<2.0

Example pyproject.toml entry:

dependencies = [
  "numpy~=2.4",
]

## Version Locking
4. Generate a lockfile with exact versions, hashes, and minimum package age:

uv lock --upgrade --exclude-newer "5 days"

This creates uv.lock with:
- Fully pinned versions
- SHA-256 hashes
- Registry source information
- Only packages published at least 5 days ago

Never edit uv.lock manually.

Example lock entry:

version = "2.4.0"
source = { registry = "https://pypi.org/simple" }
sdist = {
  url = "https://files.pythonhosted.org/packages/.../numpy-2.4.0.tar.gz",
  hash = "sha256:6e504f7b16118198f138ef31ba24d985b124c2c469fe8467007cf30fd992f934",
  size = 20685720,
  upload-time = "2025-12-20T16:18:19.023Z"
}

Re-run uv lock whenever you want to update dependencies.

## Install From Lockfile
5. Install dependencies strictly from the lockfile:

uv sync --frozen --reinstall

This:
- Installs only versions in uv.lock
- Verifies SHA hashes
- Forces reinstallation if necessary

## Security Audit
6. Audit locked dependencies for known vulnerabilities:

uv pip audit --locked

Note: This may not work reliably yet.

## Version Control
7. Commit the uv.lock file to your repository.

## CI/CD
8. Use uv in CI/CD pipelines (GitHub Actions example):

https://docs.astral.sh/uv/guides/integration/github/#using-uv-in-github-actions

## Run the Project
Execute your code with:

uv run example.py
