# Roles and permissions — always enforce, never skip

## Levels

| Who | Can |
|---|---|
| anonymous | read public rules and bundles, search, download |
| logged-in user | + create, vote, comment, favorite, propose edits |
| owner (`resource.user_id == current_user.id`) | + edit / delete their own resource |
| admin (`user.admin=True`, `current_user.is_admin()`) | everything, on every resource, + admin pages |
| role grant | `has_permission(key)`: `github.manage` (Git Manager), `ai.use`, `ai.manage`, `rule.tag_any` — admins have them all |

Owner and admin can edit/delete; nobody else. Never let a user act on another user's resource
without the `is_admin()` check. Private bundles (`access=False`) are visible to their owner, admins
and holders of the share link only — and must not leak in lists or searches.

## Python

```python
@bp.route('/thing/<int:thing_id>/delete', methods=['POST'])
@login_required
def delete_thing(thing_id):
    thing = ThingModel.get_thing(thing_id) or abort(404)
    if current_user.id != thing.user_id and not current_user.is_admin():
        return jsonify({'message': 'Forbidden', 'toast_class': 'danger-subtle'}), 403
    ...
```

Admin-only blueprints: a `before_request` returning 403 (redirect to login for anonymous), not a check
in each route. API: `@api_required`, same rules as the UI.

## Jinja

```jinja
{% if current_user.is_authenticated %}                                   logged in
{% if current_user.id == rule.user_id %}                                 owner only
{% if current_user.is_authenticated and (current_user.id == rule.user_id or current_user.is_admin()) %}  owner or admin
```

## Vue

```js
const is_admin = {{ current_user.is_admin() | tojson }}
const currentUserId = {{ current_user.id if current_user.is_authenticated else 'null' }}
const canEdit = parseInt('{{ current_user.id }}') === resource.user_id || is_admin
```

A hidden button is a convenience, not a protection: the route must refuse the request too. Tests check
every (action × who) pair — see testing.md.
