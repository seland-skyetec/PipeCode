# MVP and Implementation Sequence

## Objective

Build the smallest version that proves PipeCode as a **daily Pi-first workbench**, not merely a visual agent configurator.

The critical loop is:

```text
define Pi agents
      ↓
compose a team
      ↓
start a session with a Pi owner
      ↓
owner delegates
      ↓
observe run
      ↓
inspect result/artifact
      ↓
continue chat
```

## Phase 0 — Freeze the product model

Before backend work, retain the current frontend concepts:

- T3 Code-inspired session sidebar + chat
- Pi Agents library
- team tabs
- free vs guided teams
- Pi first-class nodes
- Codex/Claude secondary CLI nodes

Do not add more graph semantics during this phase.

### Done when

The UI can represent all core entities with mock data and no concept is ambiguous.

## Phase 1 — Real Pi owner session

Implement one real session using the official Pi SDK or RPC interface.

Scope:

- select project directory
- choose a primary/all Pi agent definition
- start/restore a Pi session
- stream messages to the chat
- send prompt / steer / stop
- persist PipeCode session metadata

No subagents required yet.

### Done when

PipeCode can replace a normal Pi terminal for one simple coding session without losing the session.

## Phase 2 — Pi Agent Library

Make the current visual Pi editor real.

Scope:

- discover project `.pi/agents/*.md`
- discover global `~/.pi/agent/agents/*.md`
- parse pi-open-agents frontmatter
- edit supported fields
- raw Markdown preview/edit path
- create/duplicate/save
- preserve project/global scope
- validate before write

Avoid inventing a second agent-definition format.

### Done when

An agent authored in PipeCode can immediately be used by Pi outside PipeCode.

## Phase 3 — Reusable teams

Persist team definitions.

Scope:

- Team Library
- Pi agent references
- Codex/Claude placeholder worker references
- free mode
- guided mode
- optional edges
- team tabs attached to a session
- attach/detach teams

Use React Flow rather than custom graph interaction code.

### Done when

A session can be configured with several reusable agent teams and the configuration survives restart.

## Phase 4 — One real delegation path

Choose one implementation and make it robust:

### Candidate path A

Owner Pi → pi-open-agents subagent

### Candidate path B

Owner Pi → Herdr-visible Pi child session

Implement:

- create delegation run
- worker state
- task/result
- parent-child relation
- completion/failure
- owner receives result
- Runs view displays trace

### Decision criterion

Choose the path that gives the cleanest combination of:

- fidelity to Pi semantics
- visibility
- steerability
- reliability
- minimal glue code

Do not choose based only on architectural elegance.

## Phase 5 — Herdr external workers

Add Codex CLI and Claude Code as secondary workers.

Scope:

- spawn through Herdr
- map lifecycle to PipeCode runtime events
- prompt
- interrupt/stop
- capture result
- inspect terminal/session when needed

Do not build full Codex or Claude configuration editors.

### Done when

A Pi owner can delegate a scoped job to one external CLI worker and receive a result through the same Runs model.

## Phase 6 — Human intervention

From a running delegation:

- open worker
- inspect current activity
- send steering message
- interrupt
- resume where supported
- stop
- notify owner of intervention

This is a key PipeCode advantage over invisible headless subagents.

## Phase 7 — Artifacts

Add a small artifact model.

Start with:

- Git diff
- changed files
- Markdown/report
- test result

Expose artifacts from both chat and Runs without embedding full content in the main transcript.

## Phase 8 — Project Context

Add an inspectable context view.

Scope:

- detected `AGENTS.md` / project instructions
- selected project docs
- default context sources
- per-session context selection

Do not build vector retrieval yet.

## Phase 9 — Versioning

Use Git for project-local definitions where practical.

Provide:

- compare
- history
- duplicate
- restore

Use lightweight application history for global definitions if needed.

## Phase 10 — Packaging / remote

Only after the local product loop is good:

- Electron or Tauri packaging
- background local service
- remote authentication
- mobile-responsive client
- optional multi-machine Herdr support

## What not to build in MVP

- generic workflow execution engine
- edge conditions
- loops
- cron
- marketplace
- multi-user collaboration
- eval platform
- enterprise tenancy
- generic provider system
- deep dashboards
- database-backed RAG

## Suggested repository shape

This is intentionally provisional:

```text
PipeCode/
├── apps/
│   ├── web/               # React frontend
│   └── server/            # local control plane
├── packages/
│   ├── domain/            # product types/invariants
│   ├── pi-adapter/
│   ├── herdr-adapter/
│   └── agent-definitions/ # pi-open-agents parse/write
├── docs/
└── AGENTS.md
```

A monorepo is useful only if these packages actually become independent seams. Do not create empty packages in advance.

## First engineering spike

The first backend spike should answer three questions with running code:

1. Can a web/local client create and stream a Pi SDK session cleanly?
2. Can PipeCode round-trip a real pi-open-agents Markdown definition without data loss?
3. Can one Pi owner delegate to one worker and produce a structured Run record?

If these work, the rest of the product has a credible foundation.
