# Rulezet AI skill

An AI agent skill that teaches a coding assistant how to develop on
[Rulezet](https://github.com/rulezet/rulezet-core): architecture, where files go, permissions,
settings, UI patterns, tests, migrations, the editor and tester submodules, the tutorial and the commit
conventions.

A skill is a folder with a `SKILL.md` (when to use it + the essentials) and reference files the
assistant opens only when it needs them. Assistants that support agent skills load it by themselves
whenever you work on Rulezet.

## Install

```bash
git clone https://github.com/rulezet/rulezet-ai-skill.git
```

Then copy the `rulezet-dev/` folder into your assistant's skills folder — for every project (the
skills folder in your home directory), or for Rulezet only (the skills folder inside your rulezet-core
checkout). For a web assistant, zip `rulezet-dev/` and upload it in its skills settings.

## Contents

```
rulezet-dev/
├── SKILL.md                  the essentials + where to look next
└── references/
    ├── architecture.md       layout, features, rules, kits, API, jobs, connectors, first start
    ├── frontend.md           templates, Vue UMD, page layout, CSS vars, shared components, RuleList
    ├── permissions.md        roles, owner/admin guards in Python, Jinja and Vue
    ├── settings.md           app/config/<key>.json settings
    ├── testing.md            tests/ and tests_new/, the 4 layers, fixtures
    ├── editor-tester.md      rulezet-editor and rulezet-tester submodules
    ├── tutorial.md           the guided tour and data-tour anchors
    ├── workflow.md           commits, branches, migrations, running, CI
    └── conventions.md        naming and layering
```

Keep it in step with the project instructions of rulezet-core: when a rule changes there, change it
here.

## License

AGPL-3.0, like Rulezet.
