---
description: Use when designing Python API client module layout and fluent client interfaces where URL paths map strictly to route.py files and route objects.
disable-model-invocation: true
---

# API Client Design

Design Python API clients with strict path-based route modules and a fluent interface inspired by the OpenAI Python SDK.

## Core Rule

Given a client routes root such as `client/routes`, append the normalized URL segments and end with `route.py`:

```text
/v1/org           -> client/routes/v1/org/route.py
/v1/org/{org_id}  -> client/routes/v1/org/_param_org_id/route.py
```

The module layout mirrors `api-server-design`. The public client interface maps the same path to chained route objects:

```python
client.v1.org.get()
client.v1.org.post(req)
client.v1.org.param_org_id(org_id).get()
client.v1.org.param_org_id(org_id).delete()
```

## Segment Rules

Static path segments keep their URL name and become:

- directories in the module path
- route object properties in the client interface

```text
/v1  -> v1/
/org -> org/
```

```python
client.v1.org
```

Dynamic path parameters use Python package-safe `_param_` directory names and public `param_` methods:

```text
/{org_id} -> _param_org_id/
```

```python
client.v1.org.param_org_id(org_id)
```

The `_param_` prefix is for module paths. The `param_` prefix is for public client methods.

## Examples

```text
/v1/org                      -> <client_routes_root>/v1/org/route.py
/v1/org/{org_id}             -> <client_routes_root>/v1/org/_param_org_id/route.py
/v1/org/{org_id}/users       -> <client_routes_root>/v1/org/_param_org_id/users/route.py
/v1/org/{org_id}/users/{id}  -> <client_routes_root>/v1/org/_param_org_id/users/_param_id/route.py
```

```python
client.v1.org.get()
client.v1.org.post(req)
client.v1.org.param_org_id(org_id).get()
client.v1.org.param_org_id(org_id).users.get()
client.v1.org.param_org_id(org_id).users.param_id(id).delete()
```

## Route Object Shape

Static route accessors should be properties, typically cached:

```python
import typing
from functools import cached_property

if typing.TYPE_CHECKING:
    from .routes.v1.route import V1Route


class Client:
    def __init__(self, *, base_url: str, auth: object | None = None) -> None:
        self.base_url = base_url
        self.auth = auth

    @cached_property
    def v1(self) -> "V1Route":
        from .routes.v1.route import V1Route

        return V1Route(client=self)
```

Route objects own their child routes and HTTP verb methods:

```python
class OrgRoute:
    def __init__(self, *, client: Client) -> None:
        self._client = client

    def param_org_id(self, org_id: str) -> OrgIdRoute:
        return OrgIdRoute(client=self._client, org_id=org_id)

    def get(
        self,
        *,
        extra_params: dict[str, object] | None = None,
        extra_headers: dict[str, str] | None = None,
    ) -> OrgListResponse:
        ...

    def post(
        self,
        req: OrgCreateRequest,
        *,
        extra_params: dict[str, object] | None = None,
        extra_headers: dict[str, str] | None = None,
    ) -> OrgResponse:
        ...
```

Dynamic route objects store the path parameter value:

```python
class OrgIdRoute:
    def __init__(self, *, client: Client, org_id: str) -> None:
        self._client = client
        self._org_id = org_id
```

## Request Models

All declared request objects should be Pydantic `BaseModel` types.

Use request objects for request bodies:

```python
client.v1.org.post(req)
```

Do not prefer loose request body keyword arguments such as:

```python
client.v1.org.post(name="Acme")
```

HTTP verb methods may accept keyword-only `extra_params` and `extra_headers` for per-call query parameters and headers.

## Path Normalization

- Treat `/v1/org` and `/v1/org/` as the same route module.
- Strip leading and trailing slashes before mapping.
- Preserve non-parameter segments literally when they are valid Python package names, such as `v1`, `v2`, and `internal`.
- Convert dynamic `{name}` to `_param_name` in module paths.
- Convert dynamic `{name}` to `param_name(value)` in the client interface.
- Keep parameter names unchanged after the `_param_` or `param_` prefix.
- Do not use bracket directory names like `[org_id]` in Python projects.
- Do not collapse two routable paths into one `route.py`.

## Scope

This skill defines API path-to-client-module mapping and the fluent route-object interface.

Do not prescribe:

- the HTTP transport library
- sync versus async behavior
- auth refresh behavior, token storage, or auth object internals
- retries, pagination, errors, logging, or telemetry
- server-side router registration

Mention constructor settings only as examples. Auth may be an object capable of token refresh, but the refresh contract is outside this skill unless the user asks for it.

## Interactive Questions

Ask only when needed to implement the next step:

- What is the client routes root, for example `client/routes`?
- What API paths should be mapped?
- Is the route class sync, async, or paired sync and async?

## Workflow

1. Identify the client routes root.
2. Identify the target API paths.
3. Normalize each path using the segment rules.
4. Create or update the path-mapped `route.py` modules.
5. Expose static child routes as properties and dynamic child routes as `param_<name>(value)` methods.
6. Keep transport, auth refresh, retries, and other non-mapping behavior out of scope unless explicitly requested.
