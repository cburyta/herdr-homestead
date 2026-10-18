# herdr-homestead

Multi-repository workspace scaffolding, health, layouts, and agent orchestration for Herdr.

> This repository is in product-definition stage. No installable release exists yet.

## Why “Homestead”

Herdr plugins overwhelmingly use the `herdr-<feature>` repository pattern. “Homestead” distinguishes this product from Herdr's individual workspace concept: it is the durable home that organizes multiple repositories, worktrees, layouts, and agents.

Planned plugin identity:

```text
Repository: cburyta/herdr-homestead
Manifest ID: cburyta.homestead
Display name: Homestead
```

## Product direction

Build one Herdr plugin that:

- creates, migrates, validates, and opens multi-repository development environments;
- presents repository, worktree, guidance, memory, and pane health;
- creates repository-aware tabs and pane layouts;
- launches supported agents in the cwd where their native context discovery works;
- supports local and remote Herdr servers without inventing a universal skill format.

## Planning documents

- [Product plan](docs/PLAN.md)
- [Herdr Homestead PRD](docs/prds/herdr-homestead-v0.1.0.md)

## Planned installation

The first tagged release is intended to install with:

```bash
herdr plugin install cburyta/herdr-homestead --ref v0.1.0
```

The command remains provisional until a manifest and tagged release exist. The PRD defines planned trust, lifecycle, compatibility, and recovery behavior.

## Status

- Community naming and Herdr platform research complete.
- Standalone v0.1.0 PRD drafted from Herdr-native requirements.
- Next step: approve remaining product gates, then produce the technical specification and Phase 0 validation tasks.

## License

[MIT](LICENSE)
