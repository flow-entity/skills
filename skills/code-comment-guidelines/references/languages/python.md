# Python docstrings

Read when writing or editing a docstring. The gate and the never-write list stay
in `SKILL.md`.

## Placement

- A docstring is the first statement inside a module, class, or function body,
  indented under the `def`/`class` line. A string literal above the declaration
  is an ordinary expression, not a docstring; `help()` and `__doc__` do not see
  it.
- Module docstring: the first statement of the module, before imports. Package
  docstring: the module docstring of `__init__.py`.
- Public modules, classes, functions, and methods of the library or public API
  surface get docstrings (PEP 257); application internals and private helpers
  only when the gate passes or a configured linter enforces them (see Toolchain).

```python
def settle(order: Order) -> bool:
    """Settle a direct-mail order and submit it for customs declaration.

    Returns False for bonded-warehouse orders; they are handled by the batch
    pipeline. Raises InvalidTaxIdError when the buyer's EORI number is invalid.
    """
    ...
```

## Shape

- PEP 257: the first line is a command phrase ending in a period ("Return the
  pathname ..."), not a signature restatement.
- Multi-line: summary line, blank line, details. Put a blank line after a class
  docstring, and closing quotes on their own line when the docstring spans lines.
- One-liners only for obvious cases. PEP 257 permits a one-liner that paraphrases
  the name for an obvious function; this skill's gate is stricter, so write a
  one-liner only when the project's doc convention requires it (a style-guide
  override). Otherwise add a constraint, or omit the docstring for a private
  helper (see Placement). One that only repeats the signature is never enough.
- PEP 8: comments are complete sentences; put two spaces before an inline
  comment.

## Content

- Summarize behavior, arguments, return value, side effects, raised exceptions,
  and call restrictions when they are not obvious.
- Document public methods and `__init__` when the class contract depends on them.
- Args/Returns/Raises sections only when they add a constraint (range, unit,
  null rule, raised condition). Match the project's style (Google, NumPy, or
  reST); do not mix them.
- Doctests are executable documentation; prefer a verified example to prose.

## Suppressions and types

- `# type: ignore[code]` and `# noqa: RULE` need a reason in the same comment.
- Do not restate types in prose; the annotation carries them.

## Toolchain

- pydocstyle or ruff `D` rules and `interrogate` can enforce coverage and style
  (D401 imperative mood). Satisfy the configured rules; they do not replace the
  gate.

## Sources

- PEP 257: https://peps.python.org/pep-0257/
- PEP 8 comments: https://peps.python.org/pep-0008/#comments
- Checked 2026-10.
