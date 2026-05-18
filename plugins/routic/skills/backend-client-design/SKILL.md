---
description: Use when designing backend data-layer client functions for CRUD, list, and iteration, where parameters and return values follow OpenAI APIs style. Language- and library-agnostic.
disable-model-invocation: true
---

# Backend Client Design

Design backend data-layer client functions — the layer that talks to a database or storage — so their parameters and return values follow OpenAI APIs style. This skill defines wire-level shape only. It does not prescribe a programming language, type system, framework, or transport.

## Operations

A resource client exposes five operations:

| Operation  | Input                                                                                   | Output                                       |
| ---------- | --------------------------------------------------------------------------------------- | -------------------------------------------- |
| `create`   | a request object carrying the new resource's fields                                     | the created resource object                  |
| `retrieve` | the resource `id`                                                                       | the resource object                          |
| `update`   | the resource `id` and a single object carrying the fields to modify (partial)           | the updated resource object                  |
| `delete`   | the resource `id`                                                                       | a deletion confirmation object               |
| `list`     | listing parameters                                                                      | a page object                                |

`update` always takes a single object holding the fields to modify. Do not flatten the modifiable fields into separate positional or keyword arguments.

## Resource Object Shape

Every resource object includes three required fields:

- `id` — string identifier. The exact format (prefix, length, character set) is left to the backend; this skill does not constrain it.
- `object` — string discriminator naming the resource type. See Object Discriminator.
- `created_at` — integer, unix epoch seconds (UTC). Do not use ISO 8601 strings or millisecond precision at this layer.

Resources may carry any additional fields beyond these three.

## Object Discriminator

`object` values use dot notation.

- Top-level resource: a single lower-case noun, e.g. `"file"`, `"assistant"`, `"thread"`.
- Sub-resource nested under a parent resource: `"parent.child"`, e.g. `"thread.message"`, `"thread.run"`.
- Deletion confirmation: `"<resource>.deleted"`, e.g. `"file.deleted"`, `"thread.message.deleted"`.

ID-cursor page objects use the literal `"list"` discriminator (see List & Pagination). Page-token page objects omit `object`.

## Delete Response Shape

`delete` returns:

```json
{
  "id": "<the deleted resource id>",
  "object": "<resource>.deleted",
  "deleted": true
}
```

`deleted` is always `true` on a successful delete. Failure surfaces through the error envelope, not as `"deleted": false`.

## List & Pagination

`list` returns a page object. Two pagination schemes are defined. Pick one per resource and stay consistent for that resource.

### ID Cursor (default)

Use when item ids are stable, safe to expose to callers, and ordered.

Listing parameters:

- `limit` — integer, maximum number of items in the page. Default value and maximum are left to the backend.
- `order` — `"asc"` or `"desc"`. Default is `"desc"`.
- `after` — string, an item `id`. Return items strictly after this id under the current `order`.
- `before` — string, an item `id`. Return items strictly before this id under the current `order`.
- Additional resource-specific filters are allowed.

Page object shape:

```json
{
  "object": "list",
  "data": [ /* resource objects */ ],
  "first_id": "<data[0].id, or null when data is empty>",
  "last_id":  "<data[-1].id, or null when data is empty>",
  "has_more": true
}
```

### Page Token (alternative)

Use when item ids are unsuitable as cursors — for example, when wrapping an external API that returns opaque tokens, or when server-side pagination state cannot be encoded as a single id.

Listing parameters:

- `limit` — integer, same semantics as above.
- `page_token` — string, an opaque token previously issued by the server. Omit on the first request.
- Additional resource-specific filters are allowed.

Page object shape:

```json
{
  "data": [ /* resource objects */ ],
  "next_page_token": "<opaque token, or null/absent when there are no more pages>"
}
```

`next_page_token` is opaque to callers. Treat it as a server-issued continuation handle and do not parse it.

### Iteration

The default `list` return is an auto-paginating wrapper: iterating it yields every item across all pages, requesting successive pages on demand.

- ID-cursor iteration: pass the current page's `last_id` as the next request's `after`, until `has_more` is `false`.
- Page-token iteration: pass the current page's `next_page_token` as the next request's `page_token`, until `next_page_token` is absent.

Callers that want single-page access read `data` directly without iterating. Implementations may also expose a non-wrapped variant that returns one page, but the default behavior is the auto-paginating wrapper.

## Error Envelope

Errors are reported as a single object:

```json
{
  "error": {
    "type": "<error category>",
    "code": "<machine-readable code, or null>",
    "message": "<human-readable explanation>",
    "param": "<offending parameter name, or null>"
  }
}
```

`type` examples: `invalid_request_error`, `authentication_error`, `permission_error`, `not_found_error`, `rate_limit_error`, `api_error`. `code` and `param` are nullable.

## Scope

This skill defines wire-level shape for CRUD, list, and iteration on a backend data-layer client.

Do not prescribe:

- the programming language, type system, or serialization library
- transport, framework, or routing
- authentication, authorization, retries, or telemetry
- `id` format (prefix, length, character set)
- `limit` default and maximum
- caching, batching, soft-delete versus hard-delete semantics
- database engine or query shape

## References

- OpenAI API reference — <https://platform.openai.com/docs/api-reference>
- paginatic — <https://github.com/allen2c/paginatic> — reference implementation of the two pagination page-object shapes described here.
