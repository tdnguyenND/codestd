# Python

> Applies to Python 3.10+. Formatter: `ruff format` (or `black`). Linter: `ruff`, with `pylint` where a
> project already runs it. Read with the [core](../core/conventions.md); IDs like `2.2` refer to it.

## Thresholds

| Signal | Flag at | Core |
|---|---|---|
| Parameters | > 5, not counting `self`/`cls`, `*args`/`**kwargs`, `_`-prefixed names or `@override`/`@overload` methods | 2.2 |
| Positional parameters | > 5 | 2.2 |
| Positional `bool` parameters | any behaviour switch | 2.3 |
| Tuple returns | ≥ 3 elements | 2.4 |
| Nesting | > 5 blocks | 3.2 |
| Cyclomatic complexity | > 10, as a suggestion; measure the repository | 2.6 |
| Function length | ~40 lines or 50 statements, soft | 2.6 |
| Module length | ~1000 lines, soft | — |
| Line length | 88, owned by the formatter | 9.1 |
| Clones | review aid only | 7.2 |

## Names (core 1)

- `snake_case` for functions and variables, `CapWords` for classes, `UPPER_CASE` for module constants.
- Acronyms stay capitalised in CapWords (`HTTPServerError`) and lowercase in snake_case
  (`send_via_https`).
- No `l`, `O` or `I` as single-character names.
- Leave the type out of the name: `id_to_name`, not `id_to_name_dict`.
- Expose a plain attribute or a `@property` instead of trivial `get_x()`/`set_x()` pairs; keep
  methods for costly or state-changing operations.
- Exception classes subclass a built-in exception and end in `Error`.
- A single leading underscore marks module or class internals.

## Functions (core 2)

- Optional, configuration and injected arguments are keyword-only: `def fit(x, *, random_state=None)`.
- Boolean switches are keyword-only: `def load(path, *, strict: bool = False)`; otherwise use an
  `Enum` or split the function.
- No mutable default arguments.

## Control flow (core 3)

- No `else` after `return`, `raise`, `break` or `continue`.

## Errors (core 4)

- No bare `except:`. `except Exception` only re-raises, logs with `logger.exception(...)`, or sits at
  an isolation boundary such as a worker loop.
- Ignore one specific exception with `contextlib.suppress(SpecificError)`, not `try/except/pass`.
- Chain causes: `raise PoolNotFoundError(pool_id) from err`.
- Keep the `try` body to the call that can raise.
- No `assert` for validation outside tests.

## Comments (core 5)

- Public modules, classes and functions have docstrings. A docstring that only restates the signature
  is rewritten with the contract: what it returns, what it raises, side effects, units (5.2).
- Where the project's pydocstyle `D1` rules are off, echo docstrings are deleted instead.
- Type hints document parameter types; describe a parameter only when its meaning is not obvious.
- TODOs link an issue: `# TODO: https://github.com/org/repo/issues/123 - drop after v2 migration`.

## Tests (core 6)

- One test function per behaviour, named `test_<unit>_<behaviour>`.
- Use `pytest.mark.parametrize` only when the bodies are identical, and name each case with
  `pytest.param(..., id="rejects_negative_amount")`.
- Plain `assert`; `pytest.raises(SpecificError, match=...)`.
- No `datetime.now()` or global randomness in tests: use fixed times, an injected clock and an
  explicit seed.

## Reuse (core 7)

- Comprehensions and built-ins over manual loops.
- Point callers at shared helpers with ruff `TID251` banned-api.

## Preferred libraries (core 7)

Use a library only when the project already declares it (`pyproject.toml`, `requirements*.txt`).
Never add a dependency without asking. The standard library comes first.

| Need | Standard library | Library, when already a dependency |
|---|---|---|
| Typed settings from env and files | — | `pydantic-settings` |
| Validating data that crosses a boundary | `dataclasses` for internal data | `pydantic` |
| Chunk, chain, group, count | `itertools` (`batched` on 3.12+), `collections` | `more-itertools` |
| Ordered dedupe | `list(dict.fromkeys(items))` | `more_itertools.unique_everseen` for unhashable items or a key |
| Decimal arithmetic (money, rates) | `decimal.Decimal` | — |
| Dates and time zones | `datetime` with `zoneinfo` | — |
| Paths | `pathlib` | — |
| HTTP client | — | `httpx`; `requests` only in sync code |
| Retries with backoff | — | `tenacity` |
| Async task groups | `asyncio.TaskGroup` (3.11+) | `anyio` |
| Tests and test doubles | `unittest.mock` | `pytest` |
| Fixed time in tests | an injected clock | `time-machine` or `freezegun` |
| Logging | `logging` | `structlog` |
| JSON | `json` | `orjson` on a measured hot path |

- Aware datetimes only: `datetime.now(timezone.utc)`, never `datetime.utcnow()`.
- Never call `requests` from `async` code; use `httpx.AsyncClient`.
- Retry only idempotent operations, and always set both a stop and a wait on `tenacity.retry`.

## Boundaries (core 8)

- Settings: a `pydantic-settings` `BaseSettings` subclass, nested models with
  `env_nested_delimiter="__"`, returned by an `@lru_cache` getter and overridden in tests. Django
  projects read `django.conf.settings`, never at import time.
- One module-level logger: `logger = logging.getLogger(__name__)`. Never log to the root logger;
  libraries add only a `NullHandler`.
- Pass log arguments instead of formatting them: `logger.info("loaded %s", pool_id)`, not an
  f-string.
- No `print` outside `__main__.py` and scripts.

## Formatting (core 9)

- `ruff format` or `black` at 88 columns.
- Imports in three groups (standard library, third-party, local), separated by blank lines and
  sorted by ruff `I`. Absolute imports; no wildcard imports.

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

## References

- [PEP 8](https://peps.python.org/pep-0008/), [PEP 257](https://peps.python.org/pep-0257/), [Logging HOWTO](https://docs.python.org/3/howto/logging.html)
- [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)
- [pydantic](https://github.com/pydantic/pydantic), [more-itertools](https://github.com/more-itertools/more-itertools), [httpx](https://github.com/encode/httpx), [requests](https://github.com/psf/requests), [tenacity](https://github.com/jd/tenacity), [anyio](https://github.com/agronholm/anyio), [time-machine](https://github.com/adamchainz/time-machine), [freezegun](https://github.com/spulec/freezegun), [structlog](https://github.com/hynek/structlog), [orjson](https://github.com/ijl/orjson)
- [ruff](https://github.com/astral-sh/ruff), [pylint](https://github.com/pylint-dev/pylint), [wemake-python-styleguide](https://github.com/wemake-services/wemake-python-styleguide), [pytest](https://docs.pytest.org/)
- [FastAPI settings guide](https://fastapi.tiangolo.com/advanced/settings/), [pydantic-settings](https://github.com/pydantic/pydantic-settings), [Django coding style](https://docs.djangoproject.com/en/dev/internals/contributing/writing-code/coding-style/)
- Project configs and contributor guides: [Home Assistant](https://github.com/home-assistant/core), [pandas](https://github.com/pandas-dev/pandas), [scikit-learn](https://github.com/scikit-learn/scikit-learn), [Apache Airflow](https://github.com/apache/airflow), [pydantic](https://github.com/pydantic/pydantic), [polars](https://github.com/pola-rs/polars), [LangChain](https://github.com/langchain-ai/langchain), [pip](https://github.com/pypa/pip)
