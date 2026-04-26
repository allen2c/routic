# Routic Skills Marketplace

This repository is a Claude Code plugin marketplace for Routic skills.

## Layout

```text
.claude-plugin/
└── marketplace.json
plugins/
└── routic/
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        └── routic/
            └── SKILL.md
```

## Local Test

From Claude Code, add this repository as a local marketplace:

```text
/plugin marketplace add .
/plugin install routic@routic
```

Then invoke the skill with:

```text
/routic
```
