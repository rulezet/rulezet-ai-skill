# Rule editor and rule tester (submodules)

## rulezet-editor — `app/modules/rulezet-editor`

Its own repository (github.com/rulezet/rulezet-editor), installed editable by `requirements.txt`;
`update.sh` / `manage.py update` float it to its remote branch, `launch.sh` / `start-dev` install it if
missing.

- `app/features/editor/` mounts its blueprint on `/editor-api` with Rulezet's own functions
  (`editor_core.py`: `validate_rule_syntax` with dependencies, rule lookup through `_active()`).
  Logged-in users only; who may use it is the setting `editor.access` (off / admins / everyone).
- The editor JS/CSS is served from `/editor-api/assets/`. Never call rulezet.org from it.
- Create page: "YARA editor" tab (`static/js/rule/YaraEditorPanel.js`, saves through
  `rule.parse_rule`). Edit page: `YaraEditRulePanel.js` (saves the Rulezet rule by its id).
- How each editor feature is wired: `app/modules/rulezet-editor/docs/rulezet/`.
- Changing the editor: edit `src/`, `npm run build` (the `dist/` files are committed), `npm test`,
  commit in the submodule, push it, then commit the new submodule pointer in rulezet-core.

## rulezet-tester — `app/modules/rulezet-tester`

Runs rules on sample sets in a **bubblewrap** sandbox (one driver per format, YARA first), with a CLI
and an HTTP API; the editor mounts it on `/editor-api/test` when given `tests=TestService(...)`.

- Needs `bubblewrap` on the server (`start-dev` probes it and offers to install it). On Ubuntu 24.04+
  unprivileged user namespaces may be blocked by AppArmor; development only:
  `RULE_TESTS_SANDBOX=process` in `.env`. CI runners usually have no working sandbox: tests must
  accept "tests unavailable".
- Who may run tests: setting `editor.test_access` (off / admins / everyone).
- Design: `app/modules/rulezet-tester/docs/design.md`; Rulezet view: `docs/future/test_panel.md`.
