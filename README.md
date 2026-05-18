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
        ├── api-server-design/
        │   └── SKILL.md
        └── backend-client-design/
            └── SKILL.md
```

## Local Test

From Claude Code, add this repository as a local marketplace:

```text
/plugin marketplace add .
/plugin install routic@routic
```

The included skills are:

- `api-server-design` — Python API server module layout where URL paths map strictly to `route.py` files.
- `api-client-design` — Python API client module layout and fluent client interface mirroring the server path map.
- `backend-client-design` — Language-agnostic backend data-layer client design (CRUD, list, iteration) following OpenAI APIs style for parameters and return values.
