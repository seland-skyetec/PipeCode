# Open-Source Candidates

> Reviewed 2026-09-28. Licenses and repository state can change; re-check before copying code into PipeCode.

This document separates:

- **dependency/integration** — use the project as an upstream component
- **selective reuse** — adapt targeted modules/components with attribution
- **reference only** — learn from architecture/UX but avoid copying code unless licensing and maintenance implications are explicitly accepted

PipeCode's own license is currently TBD.

## Recommended core

| Project | License | PipeCode use | Recommendation |
|---|---|---|---|
| [badlogic/pi-mono](https://github.com/badlogic/pi-mono) | MIT | Official Pi agent runtime, SDK/RPC, sessions, web/TUI primitives | **Core dependency / primary upstream** |
| [andrea-tomassi/pi-open-agents](https://github.com/andrea-tomassi/pi-open-agents) | MIT | Markdown/YAML agent definitions, primary/subagent modes, permissions, model/thinking, delegation guards | **Integrate and selectively reuse schema/parser logic** |
| [herdrdev/herdr](https://github.com/herdrdev/herdr) | Apache-2.0 | Persistent process/terminal runtime, heterogeneous CLI agents, lifecycle/events, multi-machine path | **Integrate through public API; avoid forking** |
| [xyflow/xyflow](https://github.com/xyflow/xyflow) | MIT | React Flow graph canvas for team nodes/optional edges | **Direct frontend dependency** |
| [pingdotgg/t3code](https://github.com/pingdotgg/t3code) | MIT | Session/chat UX, remote-ready control-surface patterns, provider/runtime architecture | **Selective UI/pattern reuse; do not make PipeCode a T3 fork** |

## Pi runtime: badlogic/pi-mono

Repository:

- https://github.com/badlogic/pi-mono

Why it matters:

Pi itself is explicitly designed to be embedded or integrated. Its coding-agent package exposes SDK, RPC and structured event modes in addition to the terminal UI.

Useful areas to inspect:

- `packages/coding-agent`
- SDK session creation/lifecycle
- session persistence
- resource/context loading
- extension loading
- model/provider abstractions
- `packages/web-ui`

What PipeCode should reuse:

- official SDK/runtime APIs
- session/event types where they fit
- native resource discovery instead of reimplementing Pi configuration

What PipeCode should own:

- project/session product model
- teams
- Runs
- artifacts
- team graph
- PipeCode persistence

**Assessment: foundational.**

## Pi agent definitions: pi-open-agents

Repository:

- https://github.com/andrea-tomassi/pi-open-agents

License: MIT.

Current agent format includes:

- `mode: primary | subagent | all`
- `hidden`
- `color`
- `model`
- `thinking`
- `systemPrompt`
- OpenCode-style `permission`
- `maxDepth`
- `allowedAgents`
- `skills`

Discovery includes project `.pi/agents/*.md` and global `~/.pi/agent/agents/*.md`.

Useful areas to inspect/reuse:

- frontmatter parsing and validation
- normalization of permissions/tools
- discovery/precedence
- subagent invocation contract
- agent visibility rules
- recursion/delegation guards

Why not rewrite immediately:

The visual editor should produce definitions that are portable and behave identically in plain Pi. Reusing upstream semantics minimizes configuration drift.

**Assessment: foundational for the Agent Library.**

## Herdr

Repository:

- https://github.com/herdrdev/herdr

License: Apache-2.0.

Useful capabilities:

- persistent terminal/process sessions
- agent lifecycle awareness
- CLI and socket API
- event subscriptions
- heterogeneous CLI agents
- remote/multi-machine model
- direct human access to running agent processes

Recommended approach:

Use Herdr as a runtime boundary rather than importing its complete UI/domain model.

PipeCode should translate Herdr events into a small internal event vocabulary.

Potential uses:

- Codex CLI worker
- Claude Code worker
- visible Pi worker sessions
- terminal inspection
- interrupt/stop
- remote execution later

**Assessment: core runtime integration, but not PipeCode's source of truth.**

## T3 Code

Repository:

- https://github.com/pingdotgg/t3code

License: MIT.

Why it matters:

PipeCode intentionally targets a similar level of everyday UX quality for sessions/chat.

Candidate areas for selective reuse or close study:

- sidebar/session interaction
- chat rendering
- composer behavior
- theme tokens
- responsive layout
- streaming message state
- reconnect/sync patterns
- remote access architecture
- source-control integration
- permission surfaces

Important boundary:

T3's main abstraction is provider-backed coding sessions. PipeCode's differentiator is the Pi Agent Library + reusable agent teams + delegation trace.

A wholesale fork would likely make upstream assumptions fight the PipeCode product model.

**Assessment: strong donor for UX/components/patterns, weak choice as the core architecture.**

## React Flow / xyflow

Repository:

- https://github.com/xyflow/xyflow

License: MIT.

Use directly for:

- draggable custom nodes
- optional edges
- pan/zoom
- fit view
- selection
- graph interaction
- later edge editing

Persist PipeCode team semantics, not React Flow's entire state object.

**Assessment: obvious dependency; do not build graph interaction from scratch.**

---

# Strong secondary candidates

## aaaxn/pi-herdr-live-agents

Repository:

- https://github.com/aaaxn/pi-herdr-live-agents

License: MIT.

What it demonstrates:

Pi subagents running as real visible Pi sessions in Herdr panes while the parent still gets tool-level spawn/wait/result semantics.

This is extremely close to PipeCode's desired "delegation + inspect/take over" experience.

Useful areas:

- spawning Pi children into Herdr
- parent/child result bridge
- mapping delegated work to visible sessions
- lifecycle cleanup
- user takeover behavior

Risk:

Small project and young codebase. Prefer extracting the bridge idea/modules over making the whole project a critical dependency until reliability is evaluated.

**Assessment: high-value integration spike.**

## AndrewJacop/pi-herdr

Repository:

- https://github.com/AndrewJacop/pi-herdr

License: MIT.

Capabilities described by the project include:

- Pi/Claude/Codex/OpenCode workers via Herdr
- spawn/message/result
- workflows
- lifecycle projection
- interrupt/resume direction
- heterogeneous completion handling

It also contains workflow runtime code ported from another MIT project with attribution.

Useful areas:

- Herdr adapter patterns
- heterogeneous worker normalization
- workflow/run lifecycle
- result collection
- model/thinking routing

Risk:

PipeCode should not inherit a workflow engine before it needs one.

**Assessment: mine the runtime glue; avoid importing the workflow worldview wholesale.**

## agegr/pi-web

Repository:

- https://github.com/agegr/pi-web

License: MIT.

Why inspect it:

- established Pi browser UI
- session management
- server/client integration
- Git/file/security logic
- PWA/web architecture
- Pi lifecycle in a web context

Possible reuse:

- Pi session streaming hooks
- model/session UI patterns
- security handling
- file interaction components

PipeCode has a different IA, so selective extraction is preferable to forking.

**Assessment: strong donor for "Pi in a browser".**

## xing-shuyin/pi-web-ui

Repository:

- https://github.com/xing-shuyin/pi-web-ui

License: MIT.

Useful features:

- browser-based Pi chat
- WebSocket streaming
- tool call display
- file tree
- terminal
- model management
- UI plugins

Potential reuse:

- chat stream protocol patterns
- terminal integration
- tool rendering
- frontend/backend protocol tests
- plugin-tab ideas later

**Assessment: strong implementation reference and selective donor.**

## n-r-w/pi-agent-suite

Repository:

- https://github.com/n-r-w/pi-agent-suite

License: MIT.

Features:

- agents
- configurable workflows
- context compression
- verification
- MCP support
- multi-model discussion

Useful for:

- studying mature Pi multi-agent edge cases
- workflow semantics if guided teams eventually need richer behavior
- independent verification concepts
- context handling

Do not adopt its full feature set into PipeCode's MVP.

**Assessment: reference now, possible targeted reuse later.**

---

# Reference with license/architecture caution

## PI-Desktop

Repository:

- https://github.com/vastsa/PI-Desktop

License: LGPL-3.0.

Why inspect it:

- large Pi-first desktop workspace
- Rust host core + Electron frontend
- projects/sessions/reviews/previews
- plugin system
- persistence patterns
- permissions and model UX

Caution:

LGPL is not the same low-friction copy/paste situation as MIT/Apache. Before copying modules into PipeCode, decide PipeCode's license and how LGPL obligations apply to the chosen integration/distribution model.

Recommended use now:

- architecture reference
- UX reference
- API boundary study

**Assessment: valuable reference; treat direct code reuse deliberately.**

## n8n

Repository:

- https://github.com/n8n-io/n8n

License: Sustainable Use License for much of the source, with separate enterprise-licensed code.

Why it matters:

PipeCode's team graph intentionally borrows the mental model of an n8n-style canvas.

Recommendation:

Use n8n as **UX inspiration only**.

Do not casually copy source code. React Flow gives us the graph primitive under MIT without pulling in n8n's product model or license constraints.

**Assessment: visual/product reference, not a donor.**

## xterm.js

Repository:

- https://github.com/xtermjs/xterm.js

License: MIT.

Use if PipeCode adds embedded terminal inspection.

Do not build a terminal emulator.

**Assessment: direct dependency when terminal UI becomes necessary.**

---

# Suggested composition

A sensible first implementation is:

```text
PipeCode frontend
├── T3-inspired session/chat shell
├── React Flow team editor
├── Pi Agent Library UI
└── Runs / Artifacts / Context

PipeCode local service
├── Pi SDK adapter              ← badlogic/pi-mono
├── Agent definition adapter    ← pi-open-agents
├── Herdr adapter               ← public socket/CLI API
└── SQLite + filesystem

Integration experiments
├── visible Pi worker bridge    ← pi-herdr-live-agents patterns
├── heterogenous worker glue    ← pi-herdr patterns
└── Pi web streaming patterns   ← pi-web / pi-web-ui
```

## Adoption priority

### Adopt early

1. Pi SDK
2. pi-open-agents format/semantics
3. React Flow
4. Herdr public runtime API

### Inspect and selectively port

1. T3 Code UI/session components
2. pi-web browser integration
3. pi-web-ui chat/tool/terminal components
4. pi-herdr-live-agents bridge
5. pi-herdr lifecycle code

### Study before adopting

1. PI-Desktop
2. pi-agent-suite workflow features
3. n8n UX

## Source reuse policy

When code is copied or substantially adapted:

1. Record repository URL.
2. Record source file/path and commit SHA.
3. Preserve required copyright/license notices.
4. Add an entry to a future `THIRD_PARTY.md`.
5. Prefer narrow modules over large vendored forks.
6. Re-check upstream license at the commit being used.

The goal is to **compose** PipeCode from proven primitives without making its architecture inseparable from any one donor project.
