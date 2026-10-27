# LLM Agent Context Log

## 2026-10-26: Repository made from `../solidcraft/`

* Copied (the old project is untouched): `pysolidcraft.py`,
  `pysolidcraft_generate.py`, `pysolidcraft_commands.json` (from
  `solidcraft-cli` 0.20.0), `examples/plate_with_holes.py`.
* The EZPkg bootstrap and PEP 723 block are gone: the module needs
  only the standard library, and a PyPI package has no use for them.
  `json`, `subprocess` and `sys` (which the bootstrap imported) are
  imported directly.  `__version__ = "0.1.0"`.
* Packaging follows `aicontexts`: `pyproject.toml` (setuptools,
  version from `pysolidcraft.__version__`, MIT-0, console script
  `pysolidcraft` = the plate demo), `LICENSE`, `git_hooks/`
  (`core.hooksPath` set), `.gitignore`, `.hgignore`.  No "Typing ::
  Typed" classifier: a single module cannot carry `py.typed`.
* Checkers: `mypy --strict` (now on in `mypy.ini`), ruff (with E501
  selected so `# noqa: E501` is not unused), flake8 and pylint all
  clean.  The generated classes keep SolidCraft's names and one
  argument per parameter, so `pysolidcraft.py` disables the matching
  pylint messages module wide (with the reason).
  `./pysolidcraft_generate.py --cached` reproduces the file exactly.
* Verified: `uv build` wheel (84 KB) into a fresh venv; the
  `pysolidcraft headless` command and the example both wrote STEP
  files (with `solidcraft-cli` from the old project's
  `squashfs-root/usr/bin` on `PATH`).
* Rules copied from `solidcraftcnc`, sources repointed at
  `../aicontexts/RULES/`.
* git and hg initialized; nothing committed to git (the user commits).
  No GitHub repository yet; the URLs in `pyproject.toml` assume
  `waynegramlich/scraftpybind`.

Next: no test suite yet; GitHub repository and workflows
(`check.yml`, `release.yml`) once there is interest.

## 2026-10-26: Moved to `~/public_html/projects/scraftpybind/`

The user meant this directory, not `~/scraftpybind/`.  Rule sources
repointed to `../aicontexts/RULES/`.
