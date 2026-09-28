# AGENTS.md

## PipeCode

PipeCode is a Pi-first agent workbench.

Read these before making structural changes:

1. `README.md`
2. `docs/PRODUCT.md`
3. `docs/ARCHITECTURE.md`
4. `docs/MVP.md`
5. `docs/OPEN_SOURCE_CANDIDATES.md`

## Product intent

The default user experience is:

```text
Project → Session → chat with one owner Pi agent
                       │
                       └→ attached agent teams
```

The app should feel like a high-quality coding-agent session client, not a workflow dashboard.

## Non-negotiable product rules

### Pi is first-class

Pi agents get:

- definition library
- visual definition editing
- primary/subagent semantics
- model/thinking settings
- permissions
- skills
- delegation guards

Codex CLI and Claude Code are external workers. Do not build equivalent configuration subsystems for them unless the product requirement changes.

### Connections are optional

A valid team may be a simple pool:

```text
A   B   C
```

It must not require edges.

A guided team may use:

```text
A → B → C
```

Do not turn every team into a workflow.

### Definition != runtime instance

Never conflate:

- Pi Agent Definition with Agent Instance
- Team Definition with Team Instance
- persisted Session with runtime process
- Run with chat message

### Chat remains primary

Delegation detail, logs and child-agent noise belong in Runs/runtime views unless they materially affect the user's main conversation.

### Upstream runtimes stay replaceable

Do not leak Herdr pane IDs, Pi child-process internals or React Flow state through core product types.

Use adapters.

## Implementation preferences

- TypeScript first while the architecture is evolving.
- Prefer React for the UI.
- Prefer official Pi SDK/RPC APIs over terminal scraping.
- Prefer React Flow over custom graph mechanics.
- Prefer Herdr's documented API over modifying/forking Herdr.
- Keep persistence boring: SQLite + user-owned files.
- Use Git for versioning project files when it already solves the problem.

## Pi agent definitions

PipeCode should round-trip definitions compatible with pi-open-agents.

Current important fields:

```yaml
name:
description:
mode: primary | subagent | all
hidden:
color:
model:
thinking:
systemPrompt: append | replace | replace-all
permission:
maxDepth:
allowedAgents:
skills:
```

Project path:

```text
.pi/agents/*.md
```

Global path:

```text
~/.pi/agent/agents/*.md
```

Do not invent a proprietary PipeCode-only Pi definition if the upstream file can represent the concept.

## Architecture guardrails

Before creating a new abstraction, ask:

1. Is this a PipeCode product concept?
2. Is this already a Pi primitive?
3. Is this already a Herdr primitive?
4. Is this only frontend rendering state?

Only (1) belongs in the core domain by default.

## Avoid premature systems

Do not add without a concrete use case:

- event sourcing infrastructure
- generic plugin SDK
- workflow DSL
- scheduler
- RAG/vector database
- distributed runtime
- provider marketplace
- multi-tenant architecture
- eval platform
- custom terminal emulator

A small typed adapter is usually preferable to a framework.

## Open-source reuse

Before copying code from another repository:

- check its current license
- record source URL/path/commit
- preserve notices
- prefer targeted reuse
- avoid copying n8n source under the assumption that it is MIT
- treat LGPL code from PI-Desktop deliberately
- create/update `THIRD_PARTY.md` once code reuse begins

MIT/Apache licensing still requires attribution/license compliance.

## Quality expectations

For implemented features:

- typecheck
- focused tests for domain invariants
- tests for Pi definition round-tripping
- tests for runtime event normalization
- no runtime ID leakage into persisted definitions
- no silent permission broadening
- meaningful error state in UI

## First vertical slice

Prioritize:

1. real Pi owner session
2. real Pi agent definition read/write
3. one reusable team
4. one real delegation
5. structured Run trace

Do not expand the feature surface until that loop works.
