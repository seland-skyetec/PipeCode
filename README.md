# PipeCode

**PipeCode is a Pi-first agent workbench for building, running, and supervising teams of coding agents from a T3 Code-like session interface.**

The core interaction is deliberately simple:

1. Open a project.
2. Start a **session** with a primary Pi agent.
3. Talk to that agent in a normal coding-chat interface.
4. Give the session one or more **agent teams**.
5. Let the primary agent delegate freely to available agents, or attach explicit connections when a task needs a guided flow.
6. Inspect delegation runs, artifacts, diffs, and agent activity without turning the main chat into a dashboard.

PipeCode is **not** intended to be a general workflow automation product. The graph is a control surface for agent availability and optional delegation structure, not a BPMN/n8n replacement.

## Product model

```text
Project
├── Project Context
├── Pi Agent Library
├── Team Library
└── Sessions
    ├── Owner / Primary Pi Agent
    ├── Attached Agent Teams
    │   ├── free delegation pools
    │   └── guided agent flows
    ├── Runs / delegation trace
    └── Artifacts
```

A session feels like T3 Code: sessions in the sidebar, a clean chat as the main surface, and contextual controls around it.

The important difference is what sits behind the chat:

```text
User
  │
  ▼
Primary Pi agent
  │
  ├── Team: General
  │   ├── Pi Implementer
  │   ├── Pi Researcher
  │   └── Claude Code CLI
  │
  ├── Team: Coding
  │   Pi Implementer → Pi Reviewer → Pi Verifier
  │
  └── Team: Research
      ├── Pi Researcher
      └── Codex CLI
```

Connections are optional. An unconnected team is simply a pool of agents the session owner may use at its own discretion.

## Pi first

Pi is the first-class agent system in PipeCode.

PipeCode should support visual authoring of Pi agent definitions compatible with **pi-open-agents**, including:

- primary / subagent / all modes
- model and thinking level
- system prompt behavior
- permissions
- skills
- delegation depth
- allowed agents
- project/global scope
- Markdown/YAML definition preview

Codex CLI and Claude Code are intentionally secondary citizens. They may be launched through the runtime layer and used as workers, but PipeCode does not need to reproduce their configuration systems.

## Runtime direction

The current architectural hypothesis is:

- **Pi / pi-mono** provides the primary agent runtime and SDK/RPC primitives.
- **pi-open-agents** provides the agent-definition and subagent semantics we expose in the GUI.
- **Herdr** provides persistent process/terminal/session primitives and a path for launching heterogeneous CLI agents.
- **PipeCode** owns product state: projects, sessions, agent teams, runs, artifacts, context and configuration.
- **React Flow** is a strong candidate for the optional team graph.
- T3 Code, Pi web clients and other projects are candidates for selective UI/runtime reuse rather than foundations we must fork.

This boundary is intentional: the frontend should not be shaped by any single upstream project's internal data model.

## Documentation

- [Product specification](docs/PRODUCT.md)
- [Architecture](docs/ARCHITECTURE.md)
- [MVP and implementation sequence](docs/MVP.md)
- [Open-source candidates and reuse strategy](docs/OPEN_SOURCE_CANDIDATES.md)
- [Agent-facing project guidance](AGENTS.md)

## Current status

Product definition / frontend prototyping.

No implementation stack is locked yet beyond the current preference for a TypeScript/React frontend and a Pi-first runtime.

## Non-goals for the first versions

- general-purpose workflow automation
- visual condition/loop programming
- scheduler/cron platform
- agent marketplace
- deep Codex/Claude configuration editors
- multi-user collaboration
- enterprise observability suite
- evaluation platform
- replacing Pi or Herdr

The first milestone is a workbench that is pleasant enough to use as the primary daily interface for Pi-based agent development.

## License

TBD. Before copying code from upstream projects, record its source and license in the repository. See [Open-source candidates](docs/OPEN_SOURCE_CANDIDATES.md).
