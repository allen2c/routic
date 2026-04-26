---
description: Use when designing API client modules that pair cleanly with Routic API server routes, favoring Pythonic, declarative, typed client interfaces.
disable-model-invocation: true
---

# API Client Design

Design API clients as small, typed interfaces that mirror server resources without copying server internals.

## Current Status

This skill is intentionally minimal until the client-side conventions are defined.

When developing it, clarify:

- Target language and HTTP client library.
- Whether clients are generated from OpenAPI or maintained by hand.
- Naming convention for resources and methods.
- Error, retry, authentication, and pagination behavior.
