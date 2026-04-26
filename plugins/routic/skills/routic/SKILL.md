---
description: Use when working on Routic tasks that benefit from step-by-step collaboration, Pythonic design, simple interfaces, and declarative configuration.
disable-model-invocation: true
---

# Routic

Guide Routic work as a short interactive loop: clarify the target, choose the smallest useful shape, implement it cleanly, then verify it.

## Working Style

- Prefer Pythonic, explicit, boring code over clever abstractions.
- Prefer declarative data structures for configuration, routing, workflows, and mappings.
- Keep interfaces small: one clear entry point, typed inputs, predictable outputs.
- Make state visible. Avoid hidden global mutation unless the host framework requires it.
- Preserve existing project conventions before introducing new patterns.
- Ask one concise question only when the next implementation step depends on the answer.

## Workflow

1. Inspect the repo before assuming architecture.
2. State the immediate next step in concrete terms.
3. Implement the smallest end-to-end change that proves the idea.
4. Add focused validation near the changed behavior.
5. Summarize what changed, how it was checked, and what decision remains.

## Design Bias

Use this shape when creating new Routic modules:

```python
from dataclasses import dataclass
from typing import Callable, Iterable


@dataclass(frozen=True)
class Step:
    name: str
    run: Callable[[], None]


def run_steps(steps: Iterable[Step]) -> None:
    for step in steps:
        step.run()
```

The exact implementation should follow the project once it exists. This example is a bias toward named, declarative steps rather than a fixed framework.

## Interaction Contract

When a user asks to build or refine Routic behavior:

- Start with the current repo facts, not a generic architecture.
- Offer a compact plan when the task has more than one meaningful step.
- Make progress in the repo unless the user is explicitly brainstorming.
- Keep questions practical and unblockable: ask for domain names, input/output examples, or acceptance criteria.
