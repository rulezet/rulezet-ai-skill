# Tests

Two suites:

- `tests/` — the current suite, run by CI on every push (`./launch.sh -t`, `.github/workflows/main.yml`).
  `conftest.py` calls `create_user_test()`, `create_admin_test()`, `create_rule_test()`
  (`app/core/utils/init_db.py`) in the `app` fixture.
- `tests_new/` — the new suite, one folder per feature. Plan and progress:
  `docs/design/test_restructure.md`. Run with `FLASKENV=testing pytest tests_new`.

Both use SQLite (`FLASKENV=testing`, no CSRF). **Never run two pytest processes in the same checkout**
(they share the SQLite file). No network: GitHub, MISP, AI providers, Vulnerability-Lookup and mail are
mocked.

## tests_new — what every feature gets (4 layers)

1. **Access control**: for each action × who (anonymous, user, owner, admin, special role), the HTTP
   status **and** the effect in the DB (a forbidden delete leaves the row; a private resource does not
   leak in lists or searches). One parametrized table per feature.
2. **CRUD**: create, read (detail + list), edit, delete, plus the feature's lifecycle (soft delete,
   trash, restore, default tags) — checked through the DB, not only the status.
3. **API**: same matrix through `/api/` with `X-API-KEY` (none, invalid, user, owner, admin key).
4. **Robustness**: too long, empty, wrong type, unicode / control chars, XSS / SQL / template injection
   strings, invalid or other users' ids, duplicates, broken rule syntax → a clean 4xx, never a 500,
   never half-written data.

Backend behaviour only — no JS, layout or CSS tests.

## Fixtures (`tests_new/conftest.py`)

- `app` — app + DB built once, every test starts from an empty, freshly seeded DB (roles/permissions,
  default tags), inside the app context; `reload(obj)` to see what a request changed.
- `users` — `SimpleNamespace(admin, owner, user)` (`helpers/users.py`); also seeds the rule formats.
- `clients` — `{"anonymous", "user", "owner", "admin"}` logged-in test clients.
- `client_as(user)` — a client for any user.
- Helpers in `tests_new/helpers/` (`rules.py: make_rule`, `bundles.py`, `inputs.py`, `access.py`…);
  copy-me examples for a new feature in `tests_new/_template/`.

```python
from tests_new.helpers.db import reload
from tests_new.helpers.rules import make_rule


def test_owner_deletes_but_user_cannot(clients, users):
    rule = make_rule(users.owner)
    assert clients["user"].post(f"/rule/delete_rule", json={"id": rule.id}).status_code == 403
    assert not reload(rule).is_deleted
```

Single test: `FLASKENV=testing pytest tests_new/rules/test_rule_crud.py -k "name"`.
