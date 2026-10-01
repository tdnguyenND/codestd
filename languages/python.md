# Python

> Applies to Python 3.10+. Formatter: `ruff format` (or `black`). Linter: `ruff`, with `pylint` where a
> project already runs it. Read with the [core](../core/conventions.md); IDs like `2.2` refer to it.

## Thresholds

| Signal | Flag at | Core | Evidence |
|---|---|---|---|
| Parameters | > 5, not counting `self`/`cls`, `*args`/`**kwargs`, `_`-prefixed names or `@override`/`@overload` methods | 2.2 | pylint `max-args` 5 ([src][pl-args]); ruff PLR0913 5 ([src][rf-args]); wemake WPS211 5 ([src][wps-args]); none of 14 large projects sets a value below 5 |
| Positional parameters | > 5 | 2.2 | ruff PLR0917 5; its documented fix is keyword-only arguments ([src][rf-pos]) |
| Positional `bool` parameters | any behaviour switch | 2.3 | ruff FBT001/FBT002 ([src][rf-fbt]); fix: keyword-only, `Enum` or split |
| Tuple returns | ≥ 3 elements | 2.4 | wemake recommends a `NamedTuple` or dataclass past 2 values ([src][wps-tuple]); mainstream linters do not measure tuple width (`max-returns` counts `return` statements) |
| Nesting | > 5 blocks | 3.2 | pylint R1702 5 ([src][pl-nest]); ruff PLR1702 5; wemake WPS220 5 |
| Cyclomatic complexity | > 10, as a suggestion; measure the repository | 2.6 | ruff C901 10 ([src][rf-cc]); projects that enforce it set 25 (Home Assistant) and 33 (pip) |
| Function length | ~40 lines or 50 statements, soft | 2.6 | Google: reconsider past ~40 lines ([guide][g-len]); pylint and ruff `max-statements` 50 |
| Module length | ~1000 lines, soft | — | pylint `max-module-lines` 1000; no sampled project enforces it |
| Line length | 88, owned by the formatter | 9.1 | ruff/black default 88 ([src][rf-line]); Django, pandas, scikit-learn, pip and polars use 88 |
| Clones | review aid only | 7.2 | none of 14 large projects gates on duplicate code; Home Assistant disables it as unavoidable |

No Python linter or style guide sets the parameter limit at 3.

## Names (core 1)

- `snake_case` for functions and variables, `CapWords` for classes, `UPPER_CASE` for module constants
  ([PEP 8][pep8-names]).
- Acronyms stay capitalised in CapWords (`HTTPServerError`) and lowercase in snake_case
  (`send_via_https`) ([PEP 8][pep8-acr], [Google][g-acr]).
- No `l`, `O` or `I` as single-character names ([PEP 8][pep8-lOI]).
- Leave the type out of the name: `id_to_name`, not `id_to_name_dict` ([Google][g-type]).
- Expose a plain attribute or a `@property` instead of trivial `get_x()`/`set_x()` pairs; keep
  methods for costly or state-changing operations ([PEP 8][pep8-acc], [Google][g-acc]).
- Exception classes subclass a built-in exception and end in `Error` ([PEP 8][pep8-exc]).
- A single leading underscore marks module or class internals.

## Functions (core 2)

- Optional, configuration and injected arguments are keyword-only: `def fit(x, *, random_state=None)`
  ([scikit-learn][skl-kw], [Airflow][af-session]).
- Boolean switches are keyword-only: `def load(path, *, strict: bool = False)`.
- No mutable default arguments ([Google][g-mutdef], ruff B006).

## Control flow (core 3)

- No `else` after `return`, `raise`, `break` or `continue` (pylint R1705, ruff RET505–RET508).

## Errors (core 4)

- No bare `except:` (ruff E722). `except Exception` (BLE001) only re-raises, logs with
  `logger.exception(...)`, or sits at an isolation boundary such as a worker loop ([ruff][rf-ble],
  [PEP 8][pep8-catch]).
- Ignore one specific exception with `contextlib.suppress(SpecificError)`, not `try/except/pass`
  (ruff SIM105, S110).
- Chain causes: `raise PoolNotFoundError(pool_id) from err` ([PEP 8][pep8-from], ruff B904).
- Keep the `try` body to the call that can raise ([PEP 8][pep8-try], [Google][g-try]).
- `assert` is not validation; it disappears under `-O` ([Google][g-assert], ruff S101 outside tests).

## Comments (core 5)

- Public modules, classes and functions have docstrings ([PEP 257][pep257-req], [Google][g-doc]). A
  docstring that only restates the signature is rewritten with the contract: what it returns, what it
  raises, side effects, units (core 5.2). PEP 257 and ruff D402 ban restating the signature.
- Where the project's pydocstyle `D1` rules are off, echo docstrings are deleted instead.
- Type hints document parameter types; describe a parameter only when its meaning is not obvious.
- TODOs link an issue: `# TODO: https://github.com/org/repo/issues/123 - drop after v2 migration`
  ([Google][g-todo]).

## Tests (core 6)

- One test function per behaviour, named `test_<unit>_<behaviour>`; pytest collects only names that
  start with `test` ([pytest][pt-collect]).
- Use `pytest.mark.parametrize` only when the bodies are identical, and name each case with
  `pytest.param(..., id="rejects_negative_amount")` ([pytest][pt-param]).
- Plain `assert`; `pytest.raises(SpecificError, match=...)` (ruff PT011).
- No `datetime.now()` or global randomness in tests: use fixed times, an injected clock and an
  explicit seed ([pandas][pd-now], [scikit-learn][skl-rng]).

## Reuse (core 7)

- Comprehensions and built-ins over manual loops (ruff C4, PERF, FURB).
- Point callers at shared helpers with ruff TID251 banned-api ([pandas][pd-banned]).

## Boundaries (core 8)

- Settings: a `pydantic-settings` `BaseSettings` subclass, nested models with
  `env_nested_delimiter="__"`, returned by an `@lru_cache` getter and overridden in tests
  ([FastAPI][fa-settings]). Django projects read `django.conf.settings`, never at import time
  ([Django][dj-settings]).
- One module-level logger: `logger = logging.getLogger(__name__)`; never log to the root logger;
  libraries add only a `NullHandler` ([logging HOWTO][py-logging]).
- Pass log arguments instead of formatting them: `logger.info("loaded %s", pool_id)`, not an
  f-string ([Google][g-log], ruff G004).
- No `print` outside `__main__.py` and scripts (ruff T201).

## Formatting (core 9)

- `ruff format` or `black` at 88 columns.
- Imports in three groups (standard library, third-party, local), separated by blank lines and
  sorted by ruff `I` ([PEP 8][pep8-imports]). Absolute imports; no wildcard imports.

## Linter settings

```toml
[tool.ruff]
line-length = 88

[tool.ruff.lint]
extend-select = [
  "B",                 # bugbear: B006 mutable defaults, B904 raise ... from
  "BLE",               # blind except
  "C4",                # comprehensions
  "C90",               # mccabe complexity
  "FBT001", "FBT002",  # positional bool parameters
  "G",                 # logging format
  "I",                 # import sorting
  "PLR0913", "PLR0917",
  "PT",                # pytest style
  "RET",               # else after return
  "S101", "S110",      # assert, try-except-pass
  "SIM",
  "T20",               # print
]

[tool.ruff.lint.per-file-ignores]
"tests/**" = ["S101", "FBT", "PLR0913"]
"scripts/**" = ["T20"]

[tool.ruff.lint.pylint]
max-args = 5
max-positional-args = 5

[tool.ruff.lint.mccabe]
max-complexity = 10
```

[pl-args]: https://github.com/pylint-dev/pylint/blob/8e75f579113a0e774d0eab2d5fee4354a56664a8/pylint/checkers/design_analysis.py#L305-L320
[pl-nest]: https://github.com/pylint-dev/pylint/blob/8e75f579113a0e774d0eab2d5fee4354a56664a8/pylint/checkers/refactoring/refactoring_checker.py#L626-L628
[rf-args]: https://github.com/astral-sh/ruff/blob/8b837319917b2ab0d16d2be23bafcb0e403005a9/crates/ruff_linter/src/rules/pylint/settings.rs#L72-L73
[rf-pos]: https://github.com/astral-sh/ruff/blob/8b837319917b2ab0d16d2be23bafcb0e403005a9/crates/ruff_linter/src/rules/pylint/rules/too_many_positional_arguments.rs#L22-L24
[rf-fbt]: https://github.com/astral-sh/ruff/blob/8b837319917b2ab0d16d2be23bafcb0e403005a9/crates/ruff_linter/src/rules/flake8_boolean_trap/rules/boolean_type_hint_positional_argument.rs#L15-L34
[rf-cc]: https://github.com/astral-sh/ruff/blob/8b837319917b2ab0d16d2be23bafcb0e403005a9/crates/ruff_linter/src/rules/mccabe/settings.rs#L12
[rf-line]: https://github.com/astral-sh/ruff/blob/8b837319917b2ab0d16d2be23bafcb0e403005a9/crates/ruff_linter/src/line_width.rs#L59-L61
[rf-ble]: https://github.com/astral-sh/ruff/blob/8b837319917b2ab0d16d2be23bafcb0e403005a9/crates/ruff_linter/src/rules/flake8_blind_except/rules/blind_except.rs#L43-L60
[wps-args]: https://github.com/wemake-services/wemake-python-styleguide/blob/d05a03bddff152feb69343c98e7e30dfd0921e2d/wemake_python_styleguide/options/defaults.py#L75
[wps-tuple]: https://github.com/wemake-services/wemake-python-styleguide/blob/d05a03bddff152feb69343c98e7e30dfd0921e2d/wemake_python_styleguide/violations/complexity.py#L1143-L1145
[pep8-names]: https://peps.python.org/pep-0008/#naming-conventions
[pep8-acr]: https://github.com/python/peps/blob/10ee163c4dc9eb82582be4534a5befb770b44201/peps/pep-0008.rst?plain=1#L932-L934
[pep8-lOI]: https://github.com/python/peps/blob/10ee163c4dc9eb82582be4534a5befb770b44201/peps/pep-0008.rst?plain=1#L983-L985
[pep8-acc]: https://github.com/python/peps/blob/10ee163c4dc9eb82582be4534a5befb770b44201/peps/pep-0008.rst?plain=1#L1148-L1152
[pep8-exc]: https://github.com/python/peps/blob/10ee163c4dc9eb82582be4534a5befb770b44201/peps/pep-0008.rst?plain=1#L1303-L1306
[pep8-from]: https://github.com/python/peps/blob/10ee163c4dc9eb82582be4534a5befb770b44201/peps/pep-0008.rst?plain=1#L1308-L1316
[pep8-catch]: https://github.com/python/peps/blob/10ee163c4dc9eb82582be4534a5befb770b44201/peps/pep-0008.rst?plain=1#L1336-L1345
[pep8-try]: https://github.com/python/peps/blob/10ee163c4dc9eb82582be4534a5befb770b44201/peps/pep-0008.rst?plain=1#L1351-L1375
[pep8-imports]: https://github.com/python/peps/blob/10ee163c4dc9eb82582be4534a5befb770b44201/peps/pep-0008.rst?plain=1#L379-L385
[pep257-req]: https://github.com/python/peps/blob/10ee163c4dc9eb82582be4534a5befb770b44201/peps/pep-0257.rst?plain=1#L50-L54
[g-len]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/pyguide.md?plain=1#L3155-L3159
[g-acr]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/pyguide.md?plain=1#L2900
[g-type]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/pyguide.md?plain=1#L2939-L2940
[g-acc]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/pyguide.md?plain=1#L2868-L2884
[g-mutdef]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/pyguide.md?plain=1#L975-L999
[g-try]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/pyguide.md?plain=1#L483-L486
[g-assert]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/pyguide.md?plain=1#L407-L413
[g-doc]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/pyguide.md?plain=1#L2062-L2091
[g-todo]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/pyguide.md?plain=1#L2691-L2723
[g-log]: https://github.com/google/styleguide/blob/4977c4e7e6bb87eea6a92eac7da6f53082bf8485/pyguide.md?plain=1#L2525-L2530
[skl-kw]: https://github.com/scikit-learn/scikit-learn/blob/bbf8863a869f118a1a42422d8cc67ec6c07f2fe0/doc/developers/develop.rst?plain=1#L112-L117
[skl-rng]: https://github.com/scikit-learn/scikit-learn/blob/bbf8863a869f118a1a42422d8cc67ec6c07f2fe0/doc/developers/develop.rst?plain=1#L719-L723
[af-session]: https://github.com/apache/airflow/blob/884e687ea7736332a3d9ea72bcae6775a9fe689c/contributing-docs/05_pull_requests.rst?plain=1#L369-L371
[pt-collect]: https://github.com/pytest-dev/pytest/blob/2887015cade4757385308e7a7d8083557fc637e2/doc/en/explanation/goodpractices.rst?plain=1#L42-L54
[pt-param]: https://github.com/pytest-dev/pytest/blob/2887015cade4757385308e7a7d8083557fc637e2/doc/en/example/parametrize.rst?plain=1#L82-L150
[pd-now]: https://github.com/pandas-dev/pandas/blob/a0806b6c9d3efb61b99ef5012e64fdb1f2f49e42/pyproject.toml#L418-L424
[pd-banned]: https://github.com/pandas-dev/pandas/blob/a0806b6c9d3efb61b99ef5012e64fdb1f2f49e42/pyproject.toml#L407
[fa-settings]: https://github.com/fastapi/fastapi/blob/33d411dbc3236275dd64d200bfe18d5d60a49b2e/docs/en/docs/advanced/settings.md?plain=1#L230-L251
[dj-settings]: https://github.com/django/django/blob/50eef95591b1e5d8c47b2e8c96066cb0c516753a/docs/internals/contributing/writing-code/coding-style.txt?plain=1#L448-L479
[py-logging]: https://github.com/python/cpython/blob/763b6edb0ec959bdfb78f098b76829365de36ca5/Doc/howto/logging.rst?plain=1#L338-L341
