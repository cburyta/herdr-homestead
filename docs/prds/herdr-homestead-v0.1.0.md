# Herdr Homestead — Product Requirements Document

**Date:** 2026-10-18

**Status:** Draft

**Version:** v0.1.0

**Product:** Installable Herdr plugin

## 1. Summary

Herdr Homestead packages multi-repository workspace creation, validation, health reporting, and agent launch workflows as an installable Herdr plugin. It uses Herdr's native workspaces, tabs, panes, worktrees, actions, and integrations to manage the operating environment around multiple repositories.

Herdr is an orchestration host, not a universal agent-resource host. Homestead therefore preserves each repository's native agent files, inventories their capabilities, and launches agents in the cwd where their own discovery rules apply. It does not pretend to provide one cross-vendor skill layer.

## 2. Problem statement

Multi-repository development requires more than a folder tree:

- Related principal checkouts need a common root.
- Development work belongs in per-repository worktrees.
- Several agents or commands may run simultaneously in distinct repository contexts.
- Users need workspace-level context and health without losing repository-native instructions.
- Terminal users benefit from persistent layouts, agent state, remote operation, and one place to see the whole system.

Herdr supplies process and layout primitives but no native multi-root project model and no cross-agent skill aggregation. A plugin must add the repository model, scaffold, validation, context index, and per-agent adapters.

## 3. Goals

1. Create, migrate, open, and audit multi-repository workspaces from Herdr.
2. Present repository, worktree, guidance, and memory health in a plugin pane.
3. Generate useful workspace/tab/pane layouts without conflating independent repositories with one Git worktree group.
4. Launch supported agent CLIs in the correct cwd.
5. Preserve child-repository instructions and skills as the source of truth.
6. Apply one deterministic scaffold and validation contract across interactive and headless plugin actions.
7. Install from a public GitHub repository with standard Herdr commands.

## 4. Non-goals for v0.1.0

- A universal skill or instruction format across agent vendors.
- Replacing Herdr's built-in integrations or lifecycle state.
- Treating several independent repositories as one native Herdr worktree group.
- Runtime registration of plugin actions or native graphical UI; Herdr plugin v1 uses manifest-declared actions and terminal panes.
- Automatically installing or modifying every supported agent's configuration.
- Starting agents before the user chooses a layout and adapters.
- Background network fetching or an always-running plugin daemon.

## 5. Target users and use cases

### Primary user

A terminal-centric developer who manages related repositories and one or more coding agents through Herdr.

### Core use cases

- Install the plugin from GitHub and invoke an action in the current Herdr workspace.
- Create a new workspace root and clone several repositories after an explicit network-operation preview.
- Preview migration of an existing folder into the standard topology.
- Open a workspace with overview and repository tabs or panes.
- Start a coordinating agent in the root and repository-scoped agents in child repositories.
- Create a task worktree for one repository without changing the others.
- Review health and drift without fetching or mutating repositories.
- Operate against a local or saved remote Herdr machine.

## 6. Product principles

- **Herdr-native orchestration:** use Herdr's CLI/socket primitives instead of terminal keystroke automation.
- **Agent-specific resources stay agent-specific:** adapters expose resources using supported mechanisms.
- **Two-phase changes:** scaffold or migrate first; open layouts and agents only after filesystem success.
- **No implicit fetch:** health reads local state unless the user explicitly requests network refresh.
- **Portable commands:** manifest entries are argv arrays and use `HERDR_BIN_PATH`.
- **Recoverable actions:** interrupted clones, partial applies, and stale panes produce actionable state.
- **Uninstall is non-destructive:** plugin removal never removes created workspaces.

## 7. Installation and discovery

The canonical manifest lives at the repository root, so the pinned install is:

```bash
herdr plugin install cburyta/herdr-homestead --ref v0.1.0
```

Development:

```bash
herdr plugin link /path/to/herdr-homestead
```

Inspection and use:

```bash
herdr plugin list --plugin cburyta.homestead
herdr plugin action list --plugin cburyta.homestead
herdr plugin log list --plugin cburyta.homestead
```

Marketplace listing requires a public repository, a valid `herdr-plugin.toml` on the default branch, and the GitHub topic `herdr-plugin`. The marketplace is unreviewed; trust documentation remains mandatory.

## 8. User experience

### 8.1 Manifest-declared actions

The plugin should expose these actions:

- `create` — choose a root and repositories, preview, scaffold, then optionally open it.
- `migrate` — inspect an existing root and preview a safe reorganization.
- `audit` — produce read-only health and validation results.
- `open-layout` — apply a selected Herdr layout to an already-valid workspace.
- `refresh-context` — regenerate the context/resource index without fetching repositories.
- `launch-agents` — start selected supported agents after required integrations and panes are verified.

Action names are qualified by the plugin ID when invoked through the CLI.

### 8.2 Plugin pane

A manifest-declared terminal pane provides:

- Workspace selection and current-context confirmation.
- Create/migrate previews.
- Repository and worktree inventory.
- Guidance, memory, and agent-resource drift.
- Layout and agent-adapter selection.
- Recovery guidance for partial operations.

The first compatible implementation may be a text UI. It must also have headless action paths that return the same structured results.

### 8.3 Suggested layouts

The plugin offers templates rather than one mandatory layout:

- **Overview:** workspace root pane plus a health pane.
- **Per repository:** one tab per repository, with shell/agent/test panes using that repository cwd.
- **Task:** selected repository worktree with agent and verification panes.
- **Cross-repo:** root-level coordinating agent plus child-repository agents.

The plugin creates topology before calling `herdr agent start`, because agent start requires an existing shell pane.

## 9. Functional requirements

### 9.1 Plugin packaging

- Include a valid, versioned `herdr-plugin.toml`.
- Declare `id`, `name`, `version`, `min_herdr_version`, description, and supported platforms.
- Declare all v0.1 actions and panes in the manifest; runtime registration is unavailable.
- Use argv command arrays, not shell strings.
- Keep editable configuration under `HERDR_PLUGIN_CONFIG_DIR`.
- Keep plugin-owned runtime state under `HERDR_PLUGIN_STATE_DIR`.
- Use `HERDR_PLUGIN_ROOT` only for installed source assets.
- Call Herdr through `HERDR_BIN_PATH` for portability.
- Pin installation examples to release tags or commits.

### 9.2 Context resolution

- Plugin commands must not assume their process cwd is the user's workspace; Herdr runs them from the plugin directory.
- Resolve invocation context from `HERDR_PLUGIN_CONTEXT_JSON` and explicit CLI parameters.
- Local and remote paths are interpreted on the machine that owns the target Herdr workspace.
- Remote actions require the plugin to be installed and enabled on the target Herdr server; plugin installation/linking is performed on that machine, not forwarded implicitly from another client.
- The UI must show the resolved machine and absolute root before applying changes.
- Actions without sufficient context must request or require an explicit root rather than guessing.

### 9.3 Scaffold and migration

Homestead must implement the product requirements in [`../PLAN.md`](../PLAN.md):

- `create`, `migrate`, and `audit` share one validation schema.
- Non-empty roots default to dry-run.
- Ambiguous changes require explicit approval.
- The workspace root remains outside Git.
- `memory/` is local-only and has no remote.
- Generated guidance inventories actual repositories.
- Existing unknown configuration is preserved.
- Re-running is idempotent.
- Partial operations produce recovery reports.

Repository cloning/fetching belongs to Homestead's workspace module, not Herdr's generic workspace abstraction. Before cloning, the preview must show normalized URL, destination, requested ref, authentication ownership, and network intent. Apply must never place credentials in argv, logs, or generated context. Existing partial destinations fail closed unless they match a recorded resumable operation. Recovery must distinguish retryable clone failure, user cancellation, authentication failure, and destination conflict. End-to-end fixtures must cover success, invalid ref, auth failure, interruption, and rerun.

### 9.4 Herdr workspace and layout integration

- A valid filesystem workspace may be opened as one Herdr workspace rooted at the workspace directory.
- Independent repositories remain independent; the plugin must not present them as one Git worktree group.
- Tabs and panes may use repository- or worktree-specific cwd values.
- Layout application must preserve user choice about focus and avoid replacing live processes without confirmation.
- Creating/opening the filesystem workspace and applying a Herdr layout are separate phases with separate results.
- Worktree creation for a selected source repository may delegate to Herdr worktree APIs only when their semantics and destination path meet workspace policy.
- If Herdr's worktree path policy cannot produce `worktrees/<repo>/<branch>`, Homestead may use Git or an explicitly selected worktree integration and then open the resulting path in Herdr. The implementation spec must validate this decision.

### 9.5 Agent adapters

- Support agent CLIs only through documented, explicit adapters and Herdr integrations.
- Report each adapter's context and resource-discovery capabilities without implying unsupported aggregation.
- Preserve each child repository's native `.pi`, `.agents`, `.claude`, `.codex`, or equivalent files.
- Start repository-scoped agents in that repository or worktree cwd so native discovery works.
- Start a coordinating agent at the workspace root only when its adapter can expose cross-repository resources safely.
- Unsupported agents may run as ordinary terminal commands and must be labeled as lacking managed skill/context integration.
- Installing an agent adapter or Herdr integration requires explicit user action; it is not a side effect of opening a workspace.

### 9.6 Generated context index

Homestead must generate an inspectable resource/context index and render it in both headless audit output and the plugin pane. The index contains:

- Repository names and absolute or workspace-relative paths.
- User-authored descriptions and dependency relationships.
- Detected instruction and skill roots by agent family.
- Common build/test commands when explicitly configured.
- Principal checkout and known worktree status.
- Generation timestamp and source manifest version.

The index must not inline secret content or blindly concatenate instruction files. Root `AGENTS.md` or vendor adapters may reference the index.

### 9.7 Health and status

The plugin pane and headless audit report:

- Dirty principal checkouts.
- Unexpected base branches.
- Local upstream divergence and last-known fetch age.
- Dirty or detached worktrees.
- Missing repository paths or remotes.
- Guidance inventory drift.
- Memory repository drift.
- Agent-resource roots and adapter readiness.
- Stale Herdr panes that reference missing paths.

Health refresh is bounded, cached when useful, and performs no implicit network fetch.

### 9.8 Event hooks and startup

- Startup hooks may restore plugin-owned presentation state and then exit; they must not act as daemons.
- Event handlers must be idempotent and tolerate duplicate or delayed invocations.
- Worktree events may refresh local inventory but must not launch agents or rewrite workspace files without explicit configuration.
- Uninstall, disable, or unlink must not leave detached background processes.

## 10. Security requirements

- Documentation must state that Herdr plugins and build commands run unsandboxed with the user's permissions.
- Install instructions must recommend reviewing source and pinning refs.
- Plugin actions must show source and destination for destructive operations.
- Credentials belong in plugin config or the user's existing agent configuration, never the plugin root, generated context, or workspace memory.
- Child instruction and skill files are untrusted inputs; indexing must not execute or concatenate them.
- Repository URLs and branch names must be passed as argv/data, not interpolated shell commands.
- Symlinks and path escapes fail closed by default.
- Marketplace listing must not be described as security review or endorsement.

## 11. Compatibility requirements

- Phase 0 must choose one of two support profiles: Herdr 0.9.3 baseline with optional-feature detection, or Herdr 0.9.5 minimum.
- Herdr 0.9.3 supports manifest plugins and ordinary panes but the installed CLI lacks `plugin update` and popup placement; Herdr 0.9.5 documents both.
- If 0.9.3 remains supported, v0.1 must avoid popup/update requirements and test install, link, disable, uninstall, ordinary panes, and minimum-version enforcement on both 0.9.3 and 0.9.5.
- If 0.9.5 is selected, set `min_herdr_version = "0.9.5"` and state clearly that 0.9.3 is unsupported.
- Linux, macOS, and Windows support declarations must reflect real tests; unsupported platforms must not be advertised.
- Cross-platform path, subprocess, Git, and line-ending behavior require test fixtures.

## 12. Suggested repository layout

```text
herdr-homestead/
├── herdr-plugin.toml
├── package.json                 # if the implementation uses Node
├── src/
│   ├── workspace/
│   ├── herdr/
│   └── ui/
├── assets/
├── test/
└── docs/
```

The repository contains one product and one release history: the Homestead Herdr plugin.

## 13. Success metrics

Homestead v0.1 succeeds against fixed empty, valid, drifted, clone-failure, stale-pane, and remote-context fixtures when:

- A user installs `cburyta/herdr-homestead` from a pinned GitHub release using standard Herdr syntax.
- An action can scaffold or audit a workspace without relying on the active shell cwd.
- The plugin opens a valid workspace and applies a selected repository-aware layout.
- Agents start only after panes exist and use the intended cwd.
- A coordinating agent launched at the workspace root receives the generated workspace index.
- An agent launched in a child repository retains its native resource discovery.
- Health detects representative repository, worktree, guidance, memory, and pane drift without fetching.
- Plugin uninstall leaves all created workspaces untouched.

## 14. Risks and mitigations

- **Scope growth:** ship scaffold, audit, layout, and explicit agent launch before adding speculative adapter automation.
- **False universal abstraction:** use explicit adapters and capability labels.
- **Version drift:** set a tested minimum and feature-detect optional APIs.
- **Partial clone/migration:** transaction journal, idempotent rerun, and recovery report.
- **Unsafe plugin execution:** pinned refs, install preview, documented trust model.
- **Layout disruption:** preview, apply only after scaffold success, and never replace live panes silently.
- **Remote path confusion:** always display target machine and resolved absolute root.
- **Async event races:** idempotent handlers and authoritative reads before mutations.

## 15. Open decisions and validation spikes

1. Choose the minimum Herdr version.
2. Validate whether Herdr worktree APIs can enforce the required destination convention.
3. Prototype a manifest action and terminal pane on 0.9.3 and 0.9.5.
4. Test target-server installation, machine-local config/state, and action context on a saved SSH machine.
5. Choose the TUI implementation language and installation/build dependency policy.
6. Decide whether the Herdr plugin remains in this repository or is split after v0.1 validation.
7. Define adapter capability metadata and how the UI explains missing skill integration.
8. Define recovery state for interrupted clone, move, layout apply, and agent launch.

## 16. Acceptance criteria

- [ ] Public GitHub source contains a valid `herdr-plugin.toml`.
- [ ] Canonical subdirectory install, link, inspect, logs, disable, and uninstall are documented; update is documented only for supported Herdr versions.
- [ ] Manifest declares honest platform and minimum-version constraints.
- [ ] Plugin commands resolve Herdr context without assuming process cwd, and remote use requires installation on the target server.
- [ ] Create, migrate, and audit use `workspace.toml` schema v1 and audit schema `homestead.audit.v1`.
- [ ] Clone previews, network approval, secret handling, failures, interruptions, and idempotent reruns meet the §9.3 contract.
- [ ] Layout application is a separate, failure-safe phase.
- [ ] Child agent configuration is preserved and never silently copied or symlinked.
- [ ] Agent capability reporting distinguishes native discovery, generated context, and unsupported aggregation.
- [ ] Unsupported agents receive an explicit generic-terminal fallback.
- [ ] Health refresh performs no implicit network fetch.
- [ ] Destructive operations show paths and require approval.
- [ ] Config and state use Herdr-provided directories.
- [ ] Uninstall leaves workspace data intact.
- [ ] Supported Herdr and operating-system combinations are tested.

## 17. Audit log

- **2026-10-18 — technical/platform review:** corrected the canonical subdirectory install, added version/capability and remote-server requirements, and expanded Phase 0 validation.
- **2026-10-18 — product/spec review:** defined product contracts, clone recovery requirements, fixed fixtures, and template-to-product ownership.
- Reviews were context-isolated within the same provider family; they were independent reviews, not cross-family verification.

## 18. Primary references

- Herdr plugins: <https://raw.githubusercontent.com/herdrdev/herdr/v0.9.5/docs/next/website/src/content/docs/plugins.mdx>
- Herdr marketplace: <https://raw.githubusercontent.com/herdrdev/herdr/v0.9.5/docs/next/website/src/content/docs/marketplace.mdx>
- Herdr integrations: <https://raw.githubusercontent.com/herdrdev/herdr/v0.9.5/docs/next/website/src/content/docs/integrations.mdx>
- Herdr agent automation: <https://raw.githubusercontent.com/herdrdev/herdr/v0.9.5/docs/next/website/src/content/docs/agent-automation.mdx>
- Herdr socket API: <https://raw.githubusercontent.com/herdrdev/herdr/v0.9.5/docs/next/website/src/content/docs/socket-api.mdx>
- Internal research input (not a portable repository dependency): `_workspaces/WORKSPACES_TEMPLATE-20261007.md` from the author's local template collection
