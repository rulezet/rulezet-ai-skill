# Architecture

## Layout

```
manage.py                  entry point for every task (start-dev, start-prod, update, deploy, test, db-init, backup…)
app.py                     CLI for the DB (-i init + first admin, -r recreate, --seed-defaults, --first-admin-password)
worker.py                  background process: job queue, telemetry, update checker, schedulers
config.py                  Development / Testing / Production configs (env vars, .env)
app/__init__.py            create_app(): extensions, every blueprint, context processors
app/core/db_class/         SQLAlchemy models, one module per feature (rule.py, bundle.py, git.py, user.py, tag.py…)
app/core/utils/            shared helpers (app_settings.py, activity_log.py, static_versions.py, init_db.py…)
app/features/<feature>/    blueprint <feature>.py (routes) + <feature>_core.py (DB logic)
app/api/<feature>/         REST API (Flask-RESTX), swagger at /api/
app/config/<key>.json      settings (see settings.md)
app/templates/<feature>/   Jinja templates, one folder per feature
app/static/js|css/<feature>/  feature JS / CSS;  app/static/js|css/components/  reusable ones
app/modules/               git submodules (rulezet-editor, rulezet-tester, rulezet-cast, misp-galaxy…)
migrations/versions/       Alembic migrations
tests/  tests_new/         test suites (see testing.md)
docs/                      design notes, refactor plan, releases, future work
```

Models are imported from their feature module: `from app.core.db_class.rule import Rule`. The package
`__init__.py` loads every module (relationships are declared by class name), so a new model module
must be added there.

## Features (`app/features/`)

`account`, `admin`, `attack`, `blog`, `bundle`, `chatbot` (Ollama-backed assistant), `community`,
`config`, `connector`, `dashboard`, `docs`, `editor` (server side of the rule editor), `jobs`, `misp`,
`notification`, `report`, `rule`, `rule_tester`, `starfield` (hidden game), `tags`, `tour` (tutorial),
`velociraptor`, `workspace`, `vulnerability` (CVE kits), `alert`, `roles`…

Logic split: the blueprint handles the request (auth, parsing, response); `*_core.py` does the work
and the DB access.

## Rules

- Soft delete: `is_deleted`, `deleted_at`, `deleted_by_id`, `delete_batch_uuid`. Trash admin at
  `/rule/trash`. Always query through `_active()` (`rule_core.py`).
- New rules get the default tags `tlp:clear` and `pap:clear` (`_attach_default_tags()`, list in
  `app/config/tags.defaults.json`).
- Creation goes through `RuleModel.add_rule_core(form_dict, user)` (syntax check with the format's own
  validator, uniqueness, risk checks).

## "Kit" pages — one subject, every rule for it

CVE response kit (`/vulnerability/<CVE|GHSA…>`, `features/vulnerability/`) and ATT&CK technique page
(`/attack/technique/<T…>`, `features/attack/attack_kit_core.py`). Shared logic (coverage per format +
gaps, chart stats, zip, bundle) in `app/features/rule/kit_core.py`, shared CSS
`css/components/rule-kit.css`, collapsible cards via `static/js/components/collapsible-cards.js`.
Expected formats per kit are settings.

## REST API

Flask-RESTX under `/api/`, CSRF-exempt, swagger at `/api/`. Namespaces per feature (public/private):
`rule`, `bundle`, `connector`, `account`, `comment`, `config`, `log`, `instance`, `rule_tester`, `utils`.
Auth with `@api_required`, key in the `X-API-KEY` header. A user's key must never allow more than the
user can do in the UI.

## Background jobs

`BackgroundJob` rows; handlers registered with `@register_handler('type')` in
`app/features/jobs/job_handlers.py` (bulk tagging, GitHub deletes, connector pulls, MISP updates,
ATT&CK parsing, trash restore…). `job_worker.py` polls every 2 s in its own process (`worker.py`,
started by `manage.py start-dev/start-prod` next to gunicorn).

## Federation and outside world

- **Connectors** (admin only): an instance pulls rules and bundles from another Rulezet instance
  (API key of an account there). Pull modes: soft (skip existing) / hard (update in place). Matching by
  UUID only — pulling twice never duplicates.
- **Rulesets mirror**: a read-only Git mirror of every published rule (`features/admin/rule_mirror/`),
  push token per mirror config.
- **Telemetry**: phone-home to rulezet.org every 24 h; `IS_OFFICIAL_INSTANCE=true` only on rulezet.org.
- **Activity log**: `from app.core.utils.activity_log import log_activity`, actions dot-namespaced
  (`rule.create`).

## First start of a new instance

`app.py -i` creates the tables and the first admin `admin@admin.admin` with a generated password and
`must_change_password=True`. Until that admin sets their own email and password, every page leads back
to the profile page, and `manage.py start-dev/start-prod` prints a fresh password in the launch
terminal at each start. On an empty instance the tutorial thanks, sends to the login, then creates a
first rule and bundle and runs the full tour.
