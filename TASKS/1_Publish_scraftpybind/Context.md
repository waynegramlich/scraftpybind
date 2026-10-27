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

## 2026-10-26: Everything working; test suite written

* There was no test suite (none in `../solidcraft` either), so
  `test_pysolidcraft.py` (stdlib `unittest`) was written: 25 tests.
  Offline: `Namespace.execute()`, every cached command reached by
  exactly one method, `connect()`, `control_dir()`, `ControlTransport`
  against a fake control port, `HeadlessTransport` against a fake
  `solidcraft-cli` script, and `pysolidcraft_generate.py --cached`
  reproducing `pysolidcraft.py` (in a scratch copy, with `ruff.toml`).
  Live (needs `solidcraft-cli` on `PATH`): an un-cached generator run
  (only for `solidcraft-cli 0.20.0`), both plates measured with
  `solidcraft-cli eval` (volumes within 1e-4; SolidCraft's are about
  5e-6 off), results and errors.  GUI: opt-in, `PYSOLIDCRAFT_TEST_PORT`.
* Fixed: the generator's default `--cli` was
  `./squashfs-root/usr/bin/solidcraft-cli` (only in `../solidcraft`);
  it is now `solidcraft-cli` on the `PATH`.
* Fixed: `HeadlessTransport.stopped()` used `poll()`, so a session
  whose output ended before it exited reported `status=None` and lost
  its error text; it now waits (up to 10 s).  The fake reproduces it.
* For the live tests:
  `PATH=../solidcraft/squashfs-root/usr/bin:$PATH python3 -m unittest`.
* The user does not expect to run live tests, so the offline tests
  are the ones that matter; the live ones stay as skips.
* `aic` (a link to `../aicontexts/ai_contexts.py`) was removed from
  git and disk; the installed `ai_contexts` is used instead.
* `ruff` 0.17.3 and `uv` 0.13.6 (updated 2026-10-26).

## 2026-10-26: Renamed `pysolidcraft` to `scraftpybind`

* At the user's request, to match the directory: `scraftpybind.py`,
  `scraftpybind_generate.py`, `scraftpybind_commands.json`,
  `test_scraftpybind.py` (all but the test with `git mv`), the PyPI
  package and console script, `SCRAFTPYBIND_TEST_PORT`,
  `scraftpybind_plate.step`.  The class `PySolidCraft` and this task's
  title are unchanged so far (asked).  The entries above use the old
  names.
* Offline `trace` coverage (`python3 -m trace --count --missing
  --module unittest test_scraftpybind`): 32 lines of 19,715 uncovered,
  in `Transport.request()`, `HeadlessTransport.stopped()`/`close()`
  timeouts, the non-`Mapping` result check, `PySolidCraft.execute()`
  and `main()`.

## 2026-10-26: `SolidCraftPyBind`; `T.pp` became `T.ps`; coverage

* The class `PySolidCraft` is now `SolidCraftPyBind` (the generator
  writes it).  This task was renamed "Publish scraftpybind", so its tag
  went from `T.pp` to `T.ps`; nothing referred to the old tag.
* `EXIT_TIMEOUT` (10 s) is how long `HeadlessTransport.stopped()` and
  `close()` wait for a session to exit; tests patch it to 0.2 s.
* New tests (29 now): `Transport.request()`, a result that is not an
  object, a session that never exits (`status=None`), and one killed
  by `close()`.  Offline `trace` coverage: 23 lines uncovered, all in
  `SolidCraftPyBind.execute()` (it sends `None` as `null`, unlike
  `Namespace.execute()`) and `main()`; the user will come back to
  both.
* Two generated docstring lines are over 72 (a long word from
  SolidCraft's text, and a long type name); left as they are.

## 2026-10-26: `# pragma: no cover` marks

* The user's convention: every line the offline tests do not reach
  ends with `# pragma: no cover`, so that `grep` finds them.  As in
  `coverage.py`, a mark on a `def`, `class`, `if` or `else:` line
  covers its block (README, Testing).
* `TestGenerator.run_generator()` now runs the generator in this
  process (patching `TARGET`, `CACHE` and `sys.argv`), so `trace` sees
  it.
* Offline `trace` coverage: `scraftpybind.py` 99.63% (23 of 6300
  uncovered), `scraftpybind_generate.py` 91.52% (42 of 495),
  `test_scraftpybind.py` 82.66% (82 of 473); the example runs only in
  the live tests.  A scratch checker (ast spans of marked lines against
  the `.cover` files) found every uncovered line marked and no stale
  mark.

## 2026-10-26: Open items 9 and 10 fixed

* 9: the generated `SolidCraftPyBind.execute()` now returns
  `Namespace(self.transport).execute(command, params)`, so it leaves
  out `None` parameters like every other method
  (`test_top_level_execute`).
* 10: `test_main` runs `main()` in this process, headless on the fake
  `solidcraft-cli` (which now answers any other command with four
  curve ids), through `LoggingHeadlessTransport`; it checks the 8
  commands, their parameters and the `Wrote ...` line.
* 31 tests.  Offline `trace`: `scraftpybind.py` 99.98% (1 of 6299
  uncovered: the `__main__` line), `scraftpybind_generate.py` 91.52%,
  `test_scraftpybind.py` 84.53%.  24 marks.

## 2026-10-26: On GitHub

* https://github.com/waynegramlich/scraftpybind, public, remote
  `github` (HTTPS), default branch `main`; `main` tracks
  `github/main`.  First push: `64fe80c` (3 commits), checked with
  `gh repo view` and `git ls-remote`; `aic` is not in the tree.
* Next: the testing and release workflows (`check.yml`,
  `release.yml`) and PyPI Trusted Publishing, per the GitHub Workflow
  Rules.

## Where things stand (2026-10-26, end of session)

* Done: the copy from `../solidcraft` works; renamed to
  `scraftpybind` (module, generator, cache, tests, package, console
  script; class `SolidCraftPyBind`; this task `T.ps`); a 31 test
  offline suite (`python3 -m unittest`, 6 live tests skipped without
  `solidcraft-cli`); all checkers clean; on GitHub (public).
* Conventions settled with the user:
  * Live tests are not run; the offline tests are what matter.
  * Every line the offline tests do not reach ends with
    `# pragma: no cover` (one mark per block, as in `coverage.py`).
    After a code change, rerun `trace` and keep the marks exact:
    `python3 -m trace --count --missing --coverdir=DIR
    --ignore-dir=$(python3 -c 'import sys;print(sys.base_prefix)')
    --module unittest test_scraftpybind`, then check every `>>>>>>`
    line in `DIR/*.cover` is inside a marked block and no mark is
    stale.  Coverage: `scraftpybind.py` 99.98% (only the `__main__`
    line), `scraftpybind_generate.py` 91.52%, `test_scraftpybind.py`
    84.53%.
  * `trace`'s `--no-report` (`-R`) drops the counts file (Python
    3.12's `trace.py:736`); subprocess tracing was tried and rejected
    as not worth it, so tests call `main()`s in-process instead.
* Not doing yet: nothing goes to pypi.org (or test.pypi.org) until
  somebody asks for the package there, so no `release.yml` and no
  Trusted Publishing for now.  A `check.yml` testing workflow
  (Linux, Windows, macOS; Python 3.11 and up) is possible meanwhile,
  if the user wants it.
* A private claude.ai page of `README.md`, published by mistake, was
  deleted at the user's request (2026-10-26).
