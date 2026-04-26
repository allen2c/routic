---
description: Use when designing Python API server route module layout where URL paths map strictly to directories and route.py files.
disable-model-invocation: true
---

# API Server Design

Design Python API server modules with strict path-based routing: every URL segment maps to one directory, and each routable path owns a local `route.py`.

## Core Rule

Given a routes root such as `app/routes`, append the normalized URL segments and end with `route.py`:

```text
/v1/org           -> app/routes/v1/org/route.py
/v1/org/{org_id}  -> app/routes/v1/org/_param_org_id/route.py
```

## Segment Rules

Static path segments keep their URL name as the directory name:

```text
/v1  -> v1/
/org -> org/
```

Dynamic path parameters use Python package-safe `_param_` directory names:

```text
/{org_id} -> _param_org_id/
/{user_id} -> _param_user_id/
```

The `_param_` prefix keeps dynamic segments visually distinct while preserving normal Python imports, static analysis, and package tooling.

Only map path segments. Ignore query strings, fragments, hosts, schemes, HTTP methods, and trailing slashes.

## Path Normalization

- Treat `/v1/org` and `/v1/org/` as the same route module.
- Strip leading and trailing slashes before mapping.
- Preserve non-parameter segments literally when they are valid Python package names, such as `v1`, `v2`, and `internal`.
- Convert dynamic `{name}` to `_param_name`.
- Keep parameter names unchanged after the `_param_` prefix.
- Do not use bracket directory names like `[org_id]` in Python projects.
- Do not collapse two routable paths into one `route.py`.

## Examples

```text
/v1/org                      -> <routes_root>/v1/org/route.py
/v1/org/{org_id}             -> <routes_root>/v1/org/_param_org_id/route.py
/v1/org/{org_id}/users       -> <routes_root>/v1/org/_param_org_id/users/route.py
/v1/org/{org_id}/users/{id}  -> <routes_root>/v1/org/_param_org_id/users/_param_id/route.py
```

## Route Module Shape

The API definitions for a path live in that path's `route.py`. For example, a FastAPI route module may look like:

```python
from fastapi import APIRouter

router = APIRouter()


@router.get("")
async def get_org() -> dict:
    return {"ok": True}
```

When showing framework examples, use an empty path (`""`) inside `route.py` because the directory path already carries the URL prefix.

## Scope

This skill only defines path-to-module mapping.

Do not prescribe:

- router registration or discovery
- schemas, services, dependencies, or business logic
- tests
- auth, validation, pagination, errors, or response shape
- framework-specific loading behavior

Mention framework code only as an example of what may live inside `route.py`.

## Interactive Questions

Ask only when needed to implement the next step:

- What is the routes root, for example `app/routes`?
- What API paths should be mapped?

## Workflow

1. Identify the routes root.
2. Identify the target API paths.
3. Normalize each path using the segment rules.
4. Create or update the path-mapped `route.py` modules.
5. Keep all non-mapping decisions out of scope unless explicitly requested.
