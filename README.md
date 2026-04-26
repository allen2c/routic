# Routic Skills Marketplace

This repository is a Claude Code plugin marketplace for Routic API design skills.

## Layout

```text
.claude-plugin/
└── marketplace.json
plugins/
└── routic/
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        ├── api-client-design/
        │   └── SKILL.md
        └── api-server-design/
            └── SKILL.md
```

## Local Test

From Claude Code, add this repository as a local marketplace:

```text
/plugin marketplace add .
/plugin install routic@routic
```

The included skills are:

- `api-server-design`
- `api-client-design`
