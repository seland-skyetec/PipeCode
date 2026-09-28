# Architecture

> Status: proposed boundaries, not a locked implementation.

## 1. Architectural objective

PipeCode should own the **product model** while delegating execution to existing runtimes.

It should avoid becoming a fork of Pi, Herdr or T3 Code.

```text
┌────────────────────────────────────────────┐
│                 PipeCode UI                │
│ Sessions · Pi Agents · Teams · Runs        │
│ Artifacts · Project Context                │
└──────────────────────┬─────────────────────┘
                       │ typed application API
┌──────────────────────▼─────────────────────┐
│            PipeCode Control Plane          │
│ persistence · events · adapters · policy   │
└────────┬─────────────────┬─────────────────┘
         │                 │
         ▼                 ▼
┌────────────────┐  ┌───────────────────────┐
│   Pi adapter   │  │     Herdr adapter     │
│ SDK / RPC      │  │ socket / CLI / events │
└───────┬────────┘  └──────────┬────────────┘
        │                      │
        ▼                      ▼
 Pi owner + Pi workers      Codex / Claude /
                           visible Pi sessions
```

## 2. Why PipeCode needs its own control plane

Upstream projects model different things:

- Pi models agent sessions and extensibility.
- pi-open-agents models reusable Pi agents and delegation.
- Herdr models persistent terminals/processes/workspaces.
- T3 Code models provider-backed coding sessions and UI state.

PipeCode adds a product domain none of these projects owns:

- projects
- owner sessions
- reusable agent teams
- optional flow edges
- run/delegation trace
- artifacts
- project context
- definition/runtime separation

These concepts should therefore be persisted by PipeCode rather than encoded into a provider-specific runtime model.

## 3. Recommended first stack

### Frontend

- React
- TypeScript
- Vite or an equivalent lightweight build setup
- React Flow (`@xyflow/react`) for team graphs
- T3 Code-inspired visual tokens and interaction density

Do not import a large UI framework solely to reproduce the current prototype.

### Local service / control plane

Prefer TypeScript/Node for the first implementation because:

- Pi's official SDK is TypeScript/Node-native.
- pi-open-agents is TypeScript.
- process and filesystem integration are straightforward.
- it minimizes language boundaries while the product model is evolving.

A desktop shell (Electron/Tauri) can be added after the web/local architecture is stable.

### Persistence

Start with SQLite plus files.

Use files when the file itself is part of the portable user contract:

- `.pi/agents/*.md`
- project instructions/context files

Use SQLite for application state:

- projects
- sessions
- teams
- team versions
- runs
- delegation events
- artifacts metadata
- runtime bindings

Do not create a distributed event store for the MVP.

## 4. Core domain model

### Project

```ts
interface Project {
  id: string;
  name: string;
  rootPath: string;
  defaultOwnerAgentId?: string;
}
```

### Pi agent definition

PipeCode should treat the Markdown definition as canonical where possible.

```ts
interface PiAgentDefinitionRef {
  id: string;
  scope: "project" | "global";
  path: string;
  name: string;
  mode: "primary" | "subagent" | "all";
  revision?: string;
}
```

The parser/editor may expose a typed representation, but round-tripping must preserve valid definitions.

### External worker reference

```ts
interface ExternalCliWorkerDefinition {
  id: string;
  kind: "codex" | "claude";
  label: string;
  defaultArgs?: string[];
}
```

### Team definition

```ts
type AgentRef =
  | { kind: "pi"; agentDefinitionId: string }
  | { kind: "external"; workerDefinitionId: string };

interface AgentTeamDefinition {
  id: string;
  projectId?: string;
  name: string;
  mode: "free" | "guided";
  nodes: TeamNode[];
  edges: TeamEdge[];
  revision: number;
}
```

Invariant:

```text
mode = free  → edges may be empty and impose no order
mode = guided → edges express intended delegation structure
```

### Session

```ts
interface Session {
  id: string;
  projectId: string;
  title: string;
  ownerAgentDefinitionId: string;
  attachedTeamRevisionIds: string[];
  status: "idle" | "running" | "stopped";
}
```

Attach a team **revision/snapshot**, not only a mutable team ID, if reproducibility matters.

### Run

```ts
interface Run {
  id: string;
  sessionId: string;
  parentRunId?: string;
  teamRevisionId?: string;
  initiatorAgentInstanceId: string;
  targetAgentInstanceId: string;
  task: string;
  state: "queued" | "running" | "blocked" | "done" | "failed" | "cancelled";
  startedAt?: string;
  endedAt?: string;
}
```

### Artifact

```ts
interface Artifact {
  id: string;
  sessionId: string;
  runId?: string;
  kind: "diff" | "file" | "report" | "plan" | "test" | "research" | "other";
  title: string;
  uri?: string;
  content?: string;
  producerAgentInstanceId?: string;
}
```

### Runtime binding

A persisted definition should not contain live process identifiers.

```ts
interface RuntimeBinding {
  agentInstanceId: string;
  runtime: "pi" | "herdr";
  runtimeSessionId?: string;
  herdrMachineId?: string;
  herdrPaneId?: string;
  pid?: number;
}
```

## 5. Pi integration

Pi provides multiple integration options:

- SDK
- RPC mode
- JSON event mode
- direct CLI

### Recommended initial experiment

Use the official Pi SDK for the **session owner** first.

Reasons:

- direct structured events
- simplest chat streaming path
- direct control over session lifecycle
- no terminal parsing
- same language as proposed control plane

Use pi-open-agents for definition/delegation semantics.

### Architecture decision to validate

There are two plausible models for delegated Pi workers:

#### A. pi-open-agents child-process execution

Pros:

- close to existing semantics
- low implementation work
- definitions already map directly

Cons:

- less visibility in Herdr unless bridged

#### B. Herdr-visible Pi workers

Pros:

- each worker becomes a real inspectable session/pane
- human takeover is natural
- heterogeneous workers share a runtime surface

Cons:

- more integration complexity
- delegation/result protocol must be robust

Do not decide this by architecture taste. Build one vertical delegation slice and compare both paths.

## 6. Herdr integration

Treat Herdr primarily as a runtime/process substrate.

Preferred integration boundary:

```ts
interface RuntimeAdapter {
  spawn(spec: SpawnSpec): Promise<RuntimeHandle>;
  send(handle: RuntimeHandle, input: RuntimeInput): Promise<void>;
  interrupt(handle: RuntimeHandle): Promise<void>;
  stop(handle: RuntimeHandle): Promise<void>;
  subscribe(handler: (event: RuntimeEvent) => void): Unsubscribe;
}
```

Herdr can initially serve:

- Codex CLI workers
- Claude Code workers
- visible Pi worker sessions
- terminal inspection
- process lifecycle
- remote-machine execution later

Do not mirror Herdr's complete workspace/tab/pane hierarchy into PipeCode's product state unless a UX feature requires it.

## 7. Event model

Use a small internal event vocabulary that normalizes runtime sources.

Examples:

```text
session.message.delta
session.message.completed

delegation.created
delegation.started
delegation.blocked
delegation.completed
delegation.failed

agent.instance.started
agent.instance.status_changed
agent.instance.stopped

artifact.created

human.intervention.sent
human.intervention.interrupted
```

Persist events that are meaningful for run history.

Do not persist every terminal byte as a domain event.

## 8. Agent definition parsing

The visual editor must support round-trip parsing for pi-open-agents Markdown/YAML definitions.

Implementation options:

1. Import/adapt its parser/schema under MIT.
2. Use a standard frontmatter/YAML parser with PipeCode validation derived from the upstream schema.

Prefer reuse of the upstream schema/normalization logic if it remains small and stable; record provenance for copied source.

The generated files must remain independently valid for Pi.

## 9. Team graph

Use React Flow or equivalent for:

- node layout
- drag/pan/zoom
- custom nodes
- optional edge creation
- selection
- fit view

The graph should store only product semantics, not React Flow's entire transient UI state.

Persist:

- node IDs
- agent references
- positions
- edges
- team mode

Avoid introducing workflow execution semantics into the graph renderer.

## 10. Runs vs chat

Chat and delegation telemetry should be separate streams.

The owner may summarize important run events into chat, but the full run tree belongs in the Runs view.

This prevents a five-agent task from turning the primary conversation into orchestration noise.

## 11. Artifacts

Artifacts should be addressable independently from messages.

Useful adapters later:

- Git diff → diff artifact
- test invocation → test artifact
- agent-written Markdown → report artifact
- research output → research artifact

Do not require agents to use a new proprietary output format before the UX is proven.

## 12. Project context

Start with explicit filesystem-backed context.

A context registry might contain:

```ts
interface ContextSource {
  id: string;
  projectId: string;
  label: string;
  kind: "file" | "directory" | "generated";
  path?: string;
  enabledByDefault: boolean;
}
```

Reuse Pi's native context discovery instead of duplicating it when possible.

PipeCode's job is to make context visible and selectable.

## 13. Security boundaries

The application may control coding agents with shell/filesystem access.

Minimum requirements:

- local binding by default
- explicit remote authentication before network exposure
- never send provider credentials to the browser
- do not expose raw Herdr control sockets directly over the network
- clearly display working directory and permission mode
- preserve Pi permission semantics rather than silently broadening them
- record human interventions and destructive runtime actions

Remote/mobile access is later work and should copy established secure patterns instead of inventing an ad hoc relay.

## 14. Desktop packaging

Do not choose Electron vs Tauri before the local web architecture works.

Electron may simplify reuse from T3/Pi web projects.

Tauri may reduce footprint and integrate well with native/local services.

This decision is reversible if the UI and local service communicate through a stable API.

## 15. Extension boundary

PipeCode itself should initially have a small adapter interface rather than a public plugin marketplace.

Potential adapters:

```text
PiRuntimeAdapter
HerdrRuntimeAdapter
CodexExternalWorker
ClaudeExternalWorker
ArtifactProvider
ContextProvider
```

A general plugin SDK is premature until two or three real adapters expose repeated needs.

## 16. Implementation rule

When an upstream project already solves a runtime primitive, integrate it rather than rebuilding it.

When PipeCode has a distinct product concept, own that concept explicitly rather than trying to encode it into the upstream runtime.

That is the main architecture boundary.
