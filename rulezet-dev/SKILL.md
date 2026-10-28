---
name: rulezet-dev
description: Everything needed to develop on Rulezet (rulezet-core — the community platform for detection rules: YARA, Sigma, Suricata, Zeek… at rulezet.org): its architecture (Flask + Vue 3 UMD + PostgreSQL), where every kind of file goes, the security and permission rules, the settings system, the UI patterns and components, the tests, migrations, background jobs, the editor/tester submodules and the commit conventions. Use it whenever you work in a rulezet-core checkout or the user mentions Rulezet, rulezet.org, rulezet-core, rulezet-editor, rulezet-tester, a Rulezet page, blueprint, rule, bundle, connector, CVE kit or ATT&CK page — adding a feature or a page, fixing a bug, writing a test or a migration, touching a template, a Vue component or a setting — even when the user does not say "Rulezet" but the code is clearly Rulezet's (app/features/, app/core/db_class/, manage.py, _active()).
---

# Developing on Rulezet

Rulezet is a community platform to share, review, test and deploy cybersecurity detection rules
(YARA, Sigma, Suricata, Zeek, CRS, Wazuh, NOVA…). Stack: **Flask + Vue.js 3 (UMD, no build step) +
PostgreSQL**. Live at rulezet.org, developed by CIRCL. This skill is the condensed developer guide:
read it before changing anything, then open the reference file for the area you touch.

## First: get oriented

```bash
python3 manage.py --help          # every command
python3 manage.py start-dev       # sets up venv, deps, migrations, then runs on 127.0.0.1:7009 (auto-reload)
python3 manage.py update          # after git pull: deps + DB in sync
python3 manage.py test            # the test suite (also ./launch.sh -t)
FLASKENV=testing pytest tests/rules/test_rule.py -k "test_name"   # one test
FLASKENV=testing pytest tests_new                                  # the new suite (by feature)
flask db migrate -m "desc" && flask db upgrade                     # migrations
```

`FLASKENV`: `development` (PostgreSQL, debug), `testing` (SQLite, no CSRF), `production`.
Secrets in `.env` (see `.env_default`). Never run two pytest processes in the same checkout: they
share the SQLite file.

## The rules that matter most

These are the ones that cause real bugs or security holes when forgotten — follow them by default.

1. **Rules are soft-deleted.** Never query `Rule.query` directly: always `_active()` from
   `app/features/rule/rule_core.py` (filters `is_deleted == False`). A raw query leaks trashed rules.
2. **Permissions, every time.** Three levels — anonymous (read public), logged-in user
   (create/vote/comment/favorite), admin (`current_user.is_admin()`), plus role grants
   (`current_user.has_permission('github.manage' | 'ai.use' | 'ai.manage' | 'rule.tag_any')`).
   Owner = `resource.user_id == current_user.id`. Owner or admin edit/delete; nobody else.
   Backend guard: `@login_required`, then
   `if current_user.id != resource.user_id and not current_user.is_admin(): return jsonify(...), 403`.
   Admin-only pages: a `before_request` returning 403, not checks scattered per route.
   Hiding a button in the template is never enough — the route must refuse too.
3. **No configuration in the code.** Lists, labels, icons, tables, prices, texts → one JSON file per
   setting in `app/config/<key>.json`, read with `setting('<key>')` / `get_setting('<key>')`. To ship a
   new default, edit the JSON — never a migration for a setting value. Only values the code branches
   on stay constants. See `references/settings.md`.
4. **Files go in their place** (one feature = same name in every folder):
   blueprint `app/features/<f>/<f>.py`, DB logic `app/features/<f>/<f>_core.py`, models
   `app/core/db_class/<f>.py` (+ register in the package `__init__.py`), REST API `app/api/<f>/`,
   templates `app/templates/<f>/`, feature JS/CSS `app/static/js/<f>/` + `app/static/css/<f>/`,
   reusable components `app/static/js/components/` + `app/static/css/components/`.
   No inline `<style>` in templates; global CSS only in `core.css`.
5. **Static files through `url_for('static', filename=…)`** (versioned, cached forever). ES modules
   keep plain imports (`import X from '/static/js/…'`) — the import map in `base.html` versions them.
6. **A component rendered once per row never fetches on mount** — share one module-level promise,
   fetch lazily (see `ReportModal.js`, `UserChip.js`).
7. **Toasts** with `create_message(msg, class)` from `/static/js/toaster.js`, never inline alert divs.
8. **Activity log** for anything worth tracing: `log_activity('feature.action', 'text', …)`.
9. **Escape user content.** Jinja autoescapes — keep it (`|safe` only on trusted HTML). In JS, build
   DOM with `textContent` or escape before `innerHTML`; Vue `v-html` only on sanitised Markdown.

## Where to look next

`references/project-rules.md` holds the **official project rules** of rulezet-core, word for word:
read it before a non-trivial change — it is the authority when anything here disagrees with it.
For the rest, open only what the task needs:

| Task | Read |
|---|---|
| The official, complete project rules | `references/project-rules.md` |
| Understand the layout, models, features, jobs, API, connectors | `references/architecture.md` |
| A page, a template, a Vue component, CSS, the UI patterns | `references/frontend.md` |
| Who may do what, guards in Python / Jinja / Vue | `references/permissions.md` |
| A list, label or text that admins may change | `references/settings.md` |
| Writing or fixing tests | `references/testing.md` |
| The rule editor / tester submodules, YARA editor | `references/editor-tester.md` |
| The Tutorial (tour) on a page, `data-tour` anchors | `references/tutorial.md` |
| Commits, branches, migrations, releases | `references/workflow.md` |
| Naming and the target code conventions | `references/conventions.md` |

## Typical recipes

**A new page** — route in the feature's blueprint, template in `app/templates/<f>/` extending
`base.html`, page banner (`.explorer-banner` with breadcrumb inside) then cards; CSS in its own file;
Vue app with delimiters `['[[', ']]']`; auth state passed from Jinja (`is_admin`, `currentUserId`).
If admin-only: add it to **both** admin navs (`macros/admin_nav.html` and the mega-menu in
`sidebar.html`). Give its key parts `data-tour="…"` anchors and, if it is a main page, a
`tour.page.<endpoint>.json` — see `references/tutorial.md`.

**A new model / column** — model in `app/core/db_class/<f>.py`, register the module in the package
`__init__.py`, then `flask db migrate -m "…"` and **read the generated file**: autogenerate picks up
unrelated drift; keep only your change. New non-null column on an existing table → `server_default`.
`manage.py start-dev` / `update` apply migrations automatically.

**A background task** — a `BackgroundJob` handler with `@register_handler('type')` in
`app/features/jobs/job_handlers.py`; the worker (`job_worker.py`, own process `worker.py`) polls every
2 s. Restart `start-dev` after changing a handler (the worker is not auto-reloaded).

**A REST endpoint** — Flask-RESTX namespace under `app/api/<f>/` (CSRF-exempt), API key auth with
`@api_required` (header `X-API-KEY`); same permission rules as the web routes.

**A setting** — `app/config/<key>.json` with `{"label","category","description","public","value"}`;
it appears on Admin → Configuration by itself. `public: true` to read it from JS at
`GET /api/config/settings/<key>`.

## Working with the maintainers

- Commit messages: `fix: [section] what changed` for features and fixes, `chg: [test section] …` for
  tests and maintenance — never `feat:`. One commit per concern (code, then its tests).
- Don't commit, push or run the whole test suite unless asked; say what you changed and how to check it.
- Prefer small, readable code that matches its neighbours (comment density, naming, idiom).
