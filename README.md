# Python example with version locking + SHA–based integrity + minimum age for dependencies
## To reduce project supply-chain risk (e.g., registry compromises like the recent Shai-Hulud incident)

### Example using UV package manager
Install requirements:  
`brew install uv` easiest for mac  

1) Run `uv init repo-name` to initialize a new project directory

2) Run `uv python install 3.14` to install a specific version of python  
    `uv python pin 3.14` to pin that version  
    `uv python upgrade 3.14` to get upgrade to latest supported versions

3) Use Broad dependencies in `pyproject.toml` only pinned to Major versions using `~=major.minor`. Human editable, but easiest with `uv add xyz`  
`uv add numpy~=2.4`  
>  dependencies = [  
>    "numpy~=2.4",

4) Run `uv lock --upgrade --exclude-newer "5 days"` to create `uv.lock` with specific version locks and SHA package validation, updated but all older than 5 days from today, rerun when you want to update. Never edit manually.  
>version = "2.4.0"  
>source = { registry = "https://pypi.org/simple" }  
>sdist = { url = "https://files.pythonhosted.org/packages/a4/7a/6a3d14e205d292b738db449d0de649b373a59edb0d0b4493821d0a3e8718/numpy-2.4.0.tar.gz", hash = "sha256:6e504f7b16118198f138ef31ba24d985b124c2c469fe8467007cf30fd992f934", size = 20685720, upload-time = "2025-12-20T16:18:19.023Z" }  

5) Run `uv sync --frozen --reinstall` installs packages from frozen lockfile forcing reinstall if needed

5) Run `uv pip audit --locked` to ensure you aren't locked to known vulnerable packages  (not working for me currenlty)

6) Commit the `uv.lock` file
 
7) In CI/CD pipelines https://docs.astral.sh/uv/guides/integration/github/#using-uv-in-github-actions


`uv run example.py` to execute


