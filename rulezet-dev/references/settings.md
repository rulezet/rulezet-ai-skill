# Settings — configuration never goes in the code

One JSON file per setting in `app/config/<key>.json`:

```json
{
  "label": "Human name",
  "category": "Editor",
  "description": "What it changes, the values it accepts.",
  "public": false,
  "value": ["…"]
}
```

Optional `"python_type": "tuple" | "set" | "dict_int_keys"`.

Read it in Python with `CONST = setting('<key>')` (live value, behaves like the dict/list; `.value` for a
real object) or `get_setting('<key>')`, both from `app/core/utils/app_settings.py`. An unknown key
raises `KeyError` — add the file.

- A new file shows up by itself on **Admin → Configuration** (`/admin/configuration`).
- Admin changes are stored in the `app_setting` table with the file value they were made from
  (`base_value`); the file stays the default and is never rewritten (keeps `git pull` clean).
- When an update changes the file, the value in use is a three-way merge (`merge_values`: new file +
  the admin's own changes); the page offers "Keep the merge" / "Use the file". Every change is in
  `app_setting_history` and can be restored.
- **To ship a new default, edit the JSON file. Never write a migration for a setting value.**
- `public: true` exposes it at `GET /api/config/settings/<key>`: JS must fetch it there instead of
  keeping its own copy. Formats: `GET /api/config/formats`.
- Only values the code branches on (parser keywords, statuses, sort columns, upload types) stay
  constants in the code.

Examples: `tags.defaults.json`, `formats.catalog.json`, `editor.access.json`, `editor.test_access.json`,
`tour.page.<endpoint>.json`, `git.org_badge.json`.
