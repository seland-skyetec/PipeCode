# Product Specification

> Status: product definition, 2026-09-28

## 1. Product thesis

PipeCode is a **Pi-first agent development workbench**.

It should feel as immediate as a modern coding-agent chat client, while making multi-agent systems understandable and controllable without forcing the user to author orchestration code.

The primary interaction is not "build a workflow". It is:

> **Talk to one capable owner agent. Give that agent a set of teams and workers. Observe, steer, and reuse the system around it.**

The owner agent is normally a Pi primary agent. It owns the session objective, decides when delegation is useful, monitors delegated work and synthesizes results back into the main conversation.

## 2. Design principles

### 2.1 Chat is the primary surface

PipeCode should open into a session-oriented coding chat, not a graph editor or operations dashboard.

The graph, runs, artifacts and runtime details are supporting surfaces.

### 2.2 Pi is the first-class agent system

PipeCode should understand Pi concepts deeply:

- primary agents
- subagents
- models
- thinking levels
- permissions
- skills
- system prompts
- delegation guards
- project/global definitions

Codex CLI and Claude Code may be available as external workers, but they are not equivalent product domains.

### 2.3 Availability and orchestration are different concepts

An agent may be available to the owner without being wired into a fixed flow.

A team therefore supports two modes:

**Free delegation**

```text
Owner
  ├── Agent A
  ├── Agent B
  └── Agent C
```

No edge semantics. The owner chooses who to use and in what order.

**Guided flow**

```text
Implementer → Reviewer → Verifier
```

Edges communicate an intended delegation structure.

The product should not assume that every useful agent system is a workflow.

### 2.4 Definitions are separate from instances

A reusable Pi agent definition is not a running agent.

A reusable team definition is not a running team.

```text
Pi Agent Definition
        │
        ├──────────────┐
        ▼              ▼
Running instance   Running instance
Session A          Session B
```

The same applies to teams.

This separation is required for versioning, reuse, session history and experimentation.

### 2.5 Hide runtime complexity until requested

The user should be able to inspect terminals, raw events and child-agent conversations, but these should not dominate normal work.

The default question PipeCode should answer is:

> What is happening, what changed, and where should I intervene?

not:

> What did every process print?

## 3. Information architecture

```text
PipeCode
│
├── Projects
│   │
│   ├── Project Context
│   ├── Pi Agent Library
│   ├── Team Library
│   └── Sessions
│       ├── Chat
│       ├── Attached Teams
│       ├── Runs
│       └── Artifacts
│
└── Global
    ├── Pi Agent Definitions
    ├── Shared Teams
    └── Runtime / Settings
```

## 4. Projects

A project is the workspace boundary for:

- repository/directory
- project-level Pi definitions
- project context
- reusable project teams
- session history
- artifacts and delegation history
- project-specific runtime settings

A project may eventually map to one or more repositories, but the first implementation can treat a project as one working directory.

## 5. Sessions

A session is the main unit of work.

Each session has:

- title
- project
- owner agent
- chat history
- attached team definitions
- runtime bindings
- run/delegation history
- artifacts
- context selection
- optional branch/worktree information

A session should resemble the mental model of a T3 Code thread, but PipeCode calls it a **session** because the owner is managing a live agent system, not only a message thread.

### 5.1 New session setup

A lightweight setup surface should allow:

```text
New session

Project
[ meeting-product ]

Owner
[ Architect (Pi) ]

Agent teams
[x] Coding Core
[x] Research
[ ] Bug Hunt
[ ] Architecture Review

Delegation
(*) Owner chooses teams freely
( ) Restrict to one team
( ) No delegation
```

Do not turn session creation into a large configuration wizard.

## 6. Pi Agent Library

PipeCode should provide a visual editor for Pi agent definitions compatible with the current `pi-open-agents` format.

A project definition can be written to:

```text
.pi/agents/*.md
```

A global definition can be written to:

```text
~/.pi/agent/agents/*.md
```

The UI should support at least:

- `name`
- `description`
- `mode: primary | subagent | all`
- `hidden`
- `color`
- `model`
- `thinking`
- `systemPrompt: append | replace | replace-all`
- `permission`
- `maxDepth`
- `allowedAgents`
- `skills`
- Markdown body / system prompt

The editor should expose both:

1. structured form controls
2. generated/raw Markdown

The raw representation remains important because the files should stay portable and usable without PipeCode.

### 6.1 Pi agent modes

- **primary** — user-facing/session-owner candidate
- **subagent** — delegation worker
- **all** — can serve either role

PipeCode should reflect these semantics in selectors rather than treating them as labels.

## 7. Team Library

A team definition is a reusable set of agent references and optional edges.

Example:

```text
Coding Core
├── Pi: Implementer
├── Pi: Reviewer
├── Pi: Verifier
└── edges:
    Implementer → Reviewer → Verifier
```

Another team may intentionally have no edges:

```text
General
├── Pi: Researcher
├── Pi: Implementer
├── Codex CLI
└── Claude Code CLI
```

### 7.1 Team node types

First-class:

- Pi agent definition reference

Secondary/external:

- Codex CLI worker
- Claude Code worker

Later adapters may add other CLI agents without changing the team model.

### 7.2 Team modes

The persisted team model should make the distinction explicit:

```ts
type TeamMode = "free" | "guided";
```

In free mode, edges should be empty or ignored.

In guided mode, edges express intended orchestration.

Edges do **not** need to become a general workflow language in early versions.

## 8. Session team tabs

The right-side session panel should use Herdr-like tabs for attached teams:

```text
[ General ] [ Coding ] [ Research ] [ + ]
```

This gives one session multiple agent configurations without forcing all workers into one graph.

The owner agent may select a team based on the current task.

## 9. Runs and delegation trace

A run is a structured record of delegated work.

Example:

```text
Run #42

Architect
│
├── Implementer
│   "Refactor session ownership"
│   ✓ done · 3m 21s
│
├── Researcher
│   "Inspect project conventions"
│   ✓ done · 1m 48s
│
└── Reviewer
    "Review implementation"
    ● working
```

A run should capture:

- initiator
- target agent
- task
- team
- parent/child relationship
- start/end times
- state
- result summary
- runtime/session reference
- artifacts produced
- human interventions

The delegation trace belongs outside the primary chat by default.

## 10. Human intervention

A user should be able to open a running worker and:

- inspect current task/state
- read recent output or conversation
- send a steering message
- interrupt
- resume
- stop
- return control to the owner

The owner must receive an event when the human intervenes, so orchestration does not continue under a false assumption about worker state.

## 11. Artifacts

Agents produce more than chat messages.

Artifacts should become first-class session objects:

- code diffs
- changed-file sets
- plans
- research notes
- test reports
- review reports
- architecture proposals
- generated documents
- terminal/log excerpts when intentionally saved

A session side panel can expose:

```text
[ Teams ] [ Runs ] [ Artifacts ]
```

Artifacts should record producer, run, timestamp and source path/reference.

## 12. Project Context

Project context should be an explicit, inspectable concept rather than invisible prompt assembly.

Potential sources:

- `AGENTS.md`
- `CLAUDE.md`
- architecture docs
- project specifications
- selected files/directories
- rules/conventions
- attached external notes

The first version does not need a vector database.

A simple selection model is enough:

```text
Always available
[x] AGENTS.md
[x] architecture.md

Optional
[ ] product-spec.md
[ ] auth-notes.md
```

Context policies may eventually be attached to teams or agent definitions.

## 13. Versioning

Agent and team definitions should become versionable.

At minimum:

- save current definition
- show change history
- compare
- restore
- duplicate/fork

The simplest implementation may use Git-backed project definitions where possible and application-level history for global definitions.

Do not build a custom version-control system if Git already provides the needed behavior.

## 14. External CLI workers

Codex and Claude Code are useful because Herdr can launch and supervise them.

PipeCode should initially model them as lightweight external workers:

```text
ExternalCliWorker
├── kind: codex | claude
├── label
├── working directory
└── runtime options
```

PipeCode should **not** initially create full visual editors for Codex/Claude settings, skills or internal configuration.

## 15. Runtime status

A small status surface should provide situational awareness without becoming an ops dashboard:

```text
main · 3 agents active · 2 delegated tasks · clean tree
```

Expanding it may show:

```text
Architect       Pi/Astra       working
Implementer     Pi/Astra       working
Reviewer        Pi/Opus        waiting
Claude Code                    idle
```

## 16. Primary UI surfaces

### Sidebar

```text
New session
Sessions
Pi agents
Teams
Project context
```

### Session

Main area:

- normal chat with owner agent
- T3 Code-inspired density and interaction model

Right panel:

- team tabs
- runs
- artifacts
- optional worker/runtime inspection

### Pi agents

- agent library
- structured editor
- raw Markdown preview
- project/global scope

### Teams

- reusable team definitions
- Pi definitions as first-class nodes
- external CLI workers as secondary nodes
- optional edges
- free/guided mode

### Project Context

- inspect and select explicit context sources

## 17. Product invariants

1. A session always has one owner.
2. The owner is normally a Pi primary/all agent.
3. Pi definitions remain portable plain files.
4. A team may contain zero edges.
5. No edge means availability, not broken configuration.
6. Codex/Claude workers never need to masquerade as Pi definitions.
7. Runtime state is separate from persisted definitions.
8. Runs and artifacts are addressable independently from chat messages.
9. Human intervention must be visible to the owner/orchestrator.
10. Upstream runtimes remain replaceable behind adapters.

## 18. Explicit non-goals

Do not add these until a concrete use case requires them:

- BPMN
- generic visual programming
- arbitrary conditional branches in the team graph
- cron/scheduling platform
- marketplace
- enterprise multi-tenancy
- generalized agent protocol
- full MCP administration UI
- automatic evaluation framework
- large observability dashboards

The risk is turning PipeCode into infrastructure before proving the workbench experience.

## 19. Success criterion

PipeCode succeeds when it becomes a better daily environment for designing and working with Pi agent systems than manually juggling:

- Pi terminals
- agent Markdown files
- Herdr panes
- delegation logs
- coding-agent CLIs
- ad hoc orchestration scripts

without removing access to those underlying primitives.
