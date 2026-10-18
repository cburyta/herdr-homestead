# Herdr Homestead — Product Plan

**Date:** 2026-10-18

**Status:** Draft for review

**Decision:** Build a standalone Herdr plugin for multi-repository workspace scaffolding and agent orchestration.

## Problem

Herdr's workspace, tab, pane, worktree, and agent primitives are intentionally general. A development environment composed of several related Git repositories needs a higher-level operating model:

- principal checkouts separated from task worktrees;
- workspace-level guidance and durable notes;
- repository-aware terminal layouts;
- health across checkouts, worktrees, guidance, and panes;
- agents launched in the cwd where their own context discovery works;
- recoverable creation and migration workflows.

Herdr does not provide a native multi-root project abstraction or a universal cross-agent skill layer. Homestead adds the multi-repository model while preserving those platform boundaries.

## Product thesis

```text
Homestead Herdr plugin
├── manifest actions
│   ├── create
│   ├── migrate
│   ├── audit
│   ├── open-layout
│   ├── refresh-context
│   └── launch-agents
├── terminal plugin pane
├── workspace module
│   ├── manifest
│   ├── scaffold and migration
│   ├── validation and health
│   └── resource/context inventory
└── Herdr integration
    ├── workspace/tab/pane layouts
    ├── worktree operations
    ├── agent adapters
    └── local and remote context
```

The plugin owns the multi-repository model and filesystem operations. Herdr remains the authority for terminal topology, agent lifecycle, and server context. Each agent remains the authority for its native instruction and skill discovery.

## Naming decision

The repository is **`cburyta/herdr-homestead`** and the planned manifest ID is **`cburyta.homestead`**.

Evidence from the Herdr marketplace:

- `herdr-<feature>` is the dominant public repository pattern.
- Generic `workspace` names are crowded and overlap Herdr's own workspace concept.
- “Homestead” communicates a durable top-level home for multiple repositories, layouts, and agents without claiming to replace Herdr workspaces.

Marketplace listing will require the `herdr-plugin` GitHub topic and a valid root `herdr-plugin.toml` on the default branch.

## Delivery sequence

### Phase 0 — Validate platform contracts

- Choose the minimum supported Herdr version after testing 0.9.3 and stable 0.9.5 capabilities.
- Prove root-manifest install, link, disable, uninstall, logs, actions, and plugin panes.
- Validate target-server installation and machine-local context on a saved SSH machine.
- Validate worktree destination semantics and `agent start` readiness requirements.
- Validate cross-platform argv and path behavior before advertising an operating system.
- Define `workspace.toml` schema v1 and audit schema `homestead.audit.v1`.
- Prototype interrupted clone and migration recovery.

### Phase 1 — Scaffold and audit

- Publish a valid `herdr-plugin.toml` at repository root.
- Implement create, migrate, and audit through one workspace contract.
- Generate concise workspace guidance and a resource/context index.
- Initialize local-only workspace memory when selected.
- Require previews and explicit approval for network and destructive operations.

### Phase 2 — Herdr layouts and agents

- Open valid filesystem workspaces in Herdr.
- Apply overview, per-repository, task, and cross-repository layouts.
- Add repository/worktree health presentation in a plugin pane.
- Launch supported agents only after pane and integration readiness checks.
- Provide a clearly labeled generic-terminal fallback.

### Phase 3 — Remote and lifecycle hardening

- Validate remote-server installation and operation end to end.
- Restore plugin-owned presentation state through bounded startup hooks.
- Add recovery for missing repositories, stale panes, interrupted clones, and partial layout application.
- Publish marketplace metadata only after supported platforms and Herdr versions are proven.

## Workspace model

```text
<workspace>/
├── AGENTS.md
├── workspace.toml
├── repos/
├── worktrees/<repo>/<branch>/
├── memory/
│   ├── README.md
│   ├── tickets/
│   ├── lessons/
│   ├── fleeting/
│   └── archive/YYYY/MM/<slug>/INDEX.md
├── scripts/
└── secrets/
```

Core invariants:

- The workspace root is not a Git repository.
- Principal checkouts live under `repos/`; development occurs in worktrees.
- Network refresh is always explicit; audit never fetches.
- `memory/` is a separate local Git repository with no remote.
- Secrets never enter generated context, logs, or plugin state.
- Existing content is preserved unless the user explicitly approves a change.
- Layout application begins only after filesystem operations succeed.
- Uninstall leaves all created workspaces untouched.

## Product boundaries

- Homestead inventories agent resources but does not translate or merge vendor-specific skills.
- Repository-scoped agents start in their repository or worktree cwd.
- A coordinating agent at the workspace root receives the generated repository/context index.
- Installing an agent integration is explicit and separate from opening a workspace.
- Independent repositories are not represented as one Git worktree group.
- Health views read local state unless the user separately requests a fetch.

## Product decisions

Decided for the first technical specification:

1. **Repository:** `cburyta/herdr-homestead`.
2. **Manifest identity:** `cburyta.homestead`, display name `Homestead`.
3. **Package shape:** one root `herdr-plugin.toml` and one release history.
4. **Workspace manifest:** `workspace.toml`, schema version 1.
5. **Audit contract:** `homestead.audit.v1`, stable ordering, and `pass`, `warn`, `fail`, and `error` findings.
6. **Refresh:** audit performs no implicit network calls.
7. **Remote model:** the plugin must be installed and enabled on the target Herdr server; paths and config are machine-local.

Remaining specification gates:

1. Minimum Herdr version: 0.9.3 baseline or 0.9.5 minimum?
2. Which operating systems have Phase 0 evidence?
3. Which agent adapters belong in v0.1 beyond generic terminal launch?
4. Can Herdr's worktree API enforce the required destination convention?
5. Which implementation language provides the smallest cross-platform install burden?

## Sources

- Internal research inputs: `_workspaces/WORKSPACES_TEMPLATE-20260825.md`, `WORKSPACES_TEMPLATE-20260922.md`, and `WORKSPACES_TEMPLATE-20261007.md` from the author's local template collection
- Herdr plugins: <https://raw.githubusercontent.com/herdrdev/herdr/v0.9.5/docs/next/website/src/content/docs/plugins.mdx>
- Herdr marketplace: <https://raw.githubusercontent.com/herdrdev/herdr/v0.9.5/docs/next/website/src/content/docs/marketplace.mdx>
- Herdr integrations: <https://raw.githubusercontent.com/herdrdev/herdr/v0.9.5/docs/next/website/src/content/docs/integrations.mdx>
- Herdr agent automation: <https://raw.githubusercontent.com/herdrdev/herdr/v0.9.5/docs/next/website/src/content/docs/agent-automation.mdx>
- Herdr socket API: <https://raw.githubusercontent.com/herdrdev/herdr/v0.9.5/docs/next/website/src/content/docs/socket-api.mdx>
- Herdr marketplace index: <https://assets.herdr.dev/plugins/index.json>
