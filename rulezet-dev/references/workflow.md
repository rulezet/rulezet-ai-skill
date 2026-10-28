# Workflow: commits, branches, migrations, releases

## Commits

- `fix: [section] what changed` — features and fixes (only the files of that change).
- `chg: [test section] …` — tests, maintenance, submodule bumps (`chg: [yara editor] rulezet-editor bumped: …`).
- Never `feat:`. One commit per concern: the code first, its tests in a second commit.
- No AI / tool attribution lines in commits or pull requests.
- Don't commit or push unless asked, and ask again each time.

## Branches

`main` is the default branch and what production runs. Work on a feature branch, push it, open a pull
request. Submodules (`rulezet-editor`, `rulezet-tester`) are committed and pushed in their own repo
first, then their new pointer is committed in rulezet-core.

## Migrations

`flask db migrate -m "desc"` then **read the generated file** (autogenerate also picks up unrelated
drift: keep only your change), then `flask db upgrade`. New non-null column on an existing table →
`server_default`. Never a migration for a setting value (edit `app/config/<key>.json`).
`manage.py start-dev` and `manage.py update` apply migrations by themselves; production runs them in
`manage.py deploy` / `start-prod` (after a backup).

## Running

- `python3 manage.py start-dev` — numbered steps (venv, dependencies when `requirements.txt` changed,
  migrations, editor/tester, sandbox probe), then gunicorn + worker.py with tidied, coloured logs.
- `python3 manage.py start-prod` / `deploy` — backup, sync with origin, deps, migrations, seed, start.
  Production log output is raw (client IPs kept).
- A second, empty instance for testing: its own DB and port, e.g.
  `DATABASE_URL=postgresql:///rulezet_test FLASK_PORT=7010 CACHE_DIR=/tmp/rz_cache python3 manage.py start-dev`
  after `createdb rulezet_test && DATABASE_URL=… python3 app.py -i && DATABASE_URL=… flask db stamp head`.

## CI

`.github/workflows/main.yml` runs `./launch.sh -t` on every push; `codeql.yml` runs CodeQL. Tests must
pass on runners without a working bubblewrap sandbox.
