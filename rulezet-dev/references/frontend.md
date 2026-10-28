# Front end: templates, Vue, CSS, components

## Basics

- Vue.js 3 **UMD** (no build step), ES modules, delimiters `['[[', ']]']` in templates.
- Static files always through `url_for('static', filename=…)` (adds `?v=<mtime>`, served `immutable`);
  never a hard-coded `/static/…` in a template. ES modules keep plain `import X from '/static/js/…'`:
  the import map in `base.html` versions them.
- A component rendered once per row (lists, tables) must not fetch on mount: one shared module-level
  promise, fetched lazily (`ReportModal.js`, `UserChip.js`).
- Toasts: `create_message(msg, class)` from `/static/js/toaster.js`.
- Pagination: `PaginationComponent` from `/static/js/rule/paginationComponent.js`.
- `ChartViewer` needs `window.echarts` (ECharts CDN) loaded globally; data:
  `{ categories: [...], series: [{ name, values: [...] }] }`.
- Tag tooltips use `<teleport to="body">` with `position:fixed` to escape `overflow:hidden` parents.
- No inline `<style>` blocks (one-off animation excepted); CSS in its own file next to the feature.

## Page layout — banner, then cards

Breadcrumb inside `.explorer-banner`, above the icon + title row. Truncate the title at 40 chars in the
breadcrumb, 60 in the `<h2>`.

```html
<div class="explorer-banner mb-4">
  <i class="fa-solid fa-[icon] banner-watermark"></i>
  <nav aria-label="breadcrumb" class="mb-2">
    <ol class="breadcrumb mb-0" style="font-size:0.75rem;text-transform:uppercase;letter-spacing:0.03em;">
      <li class="breadcrumb-item"><a href="/" class="text-decoration-none text-muted">Home</a></li>
      <li class="breadcrumb-item"><a href="/section" class="text-decoration-none text-muted">Section</a></li>
      <li class="breadcrumb-item active text-muted" aria-current="page">{% if title|length > 40 %}{{ title[:40] }}…{% else %}{{ title }}{% endif %}</li>
    </ol>
  </nav>
  <div class="d-flex align-items-center gap-3 mb-3">
    <div class="banner-icon"><i class="fa-solid fa-[icon]"></i></div>
    <div><h2 class="fw-bold mb-1">Title</h2><div class="banner-accent"></div></div>
  </div>
  <p class="text-muted mb-0" style="max-width:600px;font-size:.95rem;">Description.</p>
</div>
```

Banner gradient: blue tones only (`#0d6efd → #0a58ca`).

Main detail card (rule / bundle / user detail pages; include `css/rule/base/detail_rule.css`):

```html
<div class="card h-100 border-0 card_detail border-top shadow-lg position-relative mb-4" style="border-radius:12px;">
  <div class="card-watermark-detail"><i class="fa-solid fa-[icon]"></i></div>
  <div class="position-absolute top-0 end-0 mt-3 me-3 d-flex gap-2" style="z-index:2;"><!-- badges --></div>
  <div class="card-body p-4 p-md-5"><!-- content --></div>
</div>
```

- `.card-security-premium`: hover lift for cards in lists. `.premium-accent-line`: thin blue line on top.
- Secondary cards: `<div class="card border-0 shadow-sm rounded-3 mb-4"><div class="card-body p-4">`.
- Inline separators: `<div class="rounded-3 border p-3 mb-4" style="background:var(--light-bg-color);">`.
- Style reference for a new page: `app/templates/account/detail_user.html` + `css/account/user_detail.css`
  — KPI cards `.ud-kpi-card--blue/teal/green/gold/purple/orange`, info cells `.ud-info-grid` /
  `.ud-info-cell`, section headers `.ud-section-header` (`.ud-section-icon`, `-title`, `-sub`),
  chart cards `.ud-chart-card--accent-*`.

## Dark mode

CSS variables: `--text-color`, `--subtle-text-color` (secondary text), `--card-bg-color`,
`--border-color`, `--light-bg-color`. `var(--color-text)` does not exist.

## Shared components (`app/static/js/components/`, CSS in `css/components/`)

`SmartEditor` (code/markdown/text editor) · `CodeViewer` (props `code`, `language`, `filename`) ·
`DiffViewer` (`old_content`, `new_content`) · `ChartViewer` (line/area/bar/bar-h/pie/donut/scatter) ·
`Timeline` (`events [{date,label,icon,color}]`) · `AnsiTerminal` (`content`) · `FileTree` (`node`,
emits `select`) · `UserChip` (`user`) · `CommentThread` (`components/comments/comment-thread.js`) ·
`LoadingBar` · `KeyValue` · `DataTable` (`components/table/data-table.js`, reloads when its `fetch-url`
changes) · `LogTable` · `ReportModal` · `AttackMatrix` · `AttackDisplay` · `tour/Tour.js` (guided tour).
Feature components: `JobTracker` (`js/jobs/`), `TagInput` (`js/tags/tagInput.js`), `AttackInput`
(`js/attack/attackInput.js`), `BundleRuleSelector`, `BundleStructureEditor` (`js/bundle/`).

## RuleList (`app/static/js/rule/ruleList.js`)

Always register it with `TagsDisplaysList` + `VulnerabilityDisplaysList`. CSS deps: `dataTable.css`,
`code-viewer.css`, `ruleList.css`. Modes `read`, `select`, `manage`; bulk actions
delete/download/bundle are internal. Props `:current-user-id`, `:current-user-is-admin`,
`:current-user-is-authenticated` (edit/delete buttons then appear for owner or admin). ATT&CK chips:
`AttackDisplayList` (`js/attack/attackDisplayList.js`, CSS in `ruleList.css`). `MultiAttackFilter`'s
`apiEndpoint` must return `{ id: "T1068", name, tactic_keys, count }` — the technique id string, not
the integer PK.

## CommentThread props

`:can-create`, `:can-edit-own`, `:can-delete-own` (all = `is_authenticated`), `:can-moderate`
(= `is_admin()`).

## Admin navigation

Two navs, hand-duplicated, to update together when adding or moving an admin page:
`app/templates/macros/admin_nav.html` (inline quick-nav on admin pages) and the Admin mega-menu in
`app/templates/sidebar.html`. Same three columns (Community / Content / System).
