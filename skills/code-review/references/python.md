# Python Conventions

**Detection triggers**: presence of `pyproject.toml`, `requirements.txt`, or `setup.py`/`setup.cfg`. Version read from `pyproject.toml` (`project.requires-python` or `[tool.poetry.dependencies].python`) or from `setup.cfg` (`python_requires`).

- Follow PEP 8; rely on the project's configured formatter/linter (`black`, `ruff`, `flake8`) rather than restating its rules manually if a config file is present.
- Use type hints on function signatures and public APIs; prefer the built-in generics (`list[str]`, `X | None`) over `typing.List`/`typing.Optional` on Python ≥ 3.9/3.10 targets — check the declared minimum version before requiring the newer syntax.
- Favor `dataclasses` or Pydantic models over plain dicts/tuples for structured data crossing function boundaries.
- Never use a mutable default argument (`def f(x=[])`); default to `None` and initialize inside the function.
- Prefer f-strings over `%`-formatting or `.format()`.
- Use `pathlib.Path` over `os.path` for filesystem paths.
- Catch specific exceptions; avoid bare `except:` and avoid silently swallowing errors.
- Use context managers (`with`) for resources (files, locks, connections) instead of manual acquire/release.
- Avoid wildcard imports (`from module import *`).
- For async code, keep I/O-bound work under `async`/`await` consistently; avoid blocking calls inside async functions.
- Docstrings on public modules/classes/functions, in the style already used in the project (Google or NumPy) — do not introduce a third style.

## Performance and scalability

- Prefer generators and iterators over materializing full lists when the sequence can be large; the same applies to `.all()`-style ORM calls used only to iterate once.
- Watch for ORM N+1: `select_related`/`prefetch_related` (Django), `selectinload`/`joinedload` (SQLAlchemy). A loop that touches a related attribute is the usual tell.
- Blocking calls inside `async` functions are covered above; they are also the most common scalability defect in async Python, so report them under that angle too.
