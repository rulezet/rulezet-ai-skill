# Code conventions

The refactor plan in `docs/refactor/` (draft) describes where the code is going; new code should
already read that way. Today's layout (`<feature>.py` + `<feature>_core.py`) stays valid — don't move
files as a side effect of a fix.

## Goal

Someone opening the project must be able to **find** where something is from its name alone
(feature → folder → file), **understand** what a function does from its name, and **follow** a request
from the URL to the database through the same layers every time.

## Layers

```
route (blueprint) / api  →  logic (*_core.py, later service.py)  →  DB access  →  models
```

A route never builds a query; DB code never knows about `request` or `current_user`. Permission
checks in routes (who is calling), business rules in the logic (what is allowed).

## Names

- English only; `snake_case` in Python, `camelCase` in JS, `kebab-case` for URLs and static file names.
- Verb + object, and say what it returns: `get_rule(id)` → one or None, `list_rules(filters)` → list,
  `count_rules()` → int, `create_ / update_ / delete_ / restore_`, `is_ / has_ / can_` → bool.
- Avoid vague words alone (`utils`, `helper`, `manager`, `data`, `handle`, `process`).
- One concept, one word: `git` when any host is meant (not `github`), `rule`, `source`.

## Comments and style

- Match the surrounding code: its comment density, naming and idiom.
- Comments say why, not what. Docstrings on public functions: what it returns, what it refuses.
- Small functions, early returns, no dead code, no commented-out blocks.
- Keep the code readable for other contributors: CSS in its own file, tests next to their feature.
