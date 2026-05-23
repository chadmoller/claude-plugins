---
name: parallel-dev:init-config
version: 1.0.0
description: This skill should be used when the user wants to "set up parallel-dev", "configure parallel-dev", "initialize parallel-dev config", "add parallel-dev to this project", or wants to create a .claude/parallel-dev.local.md configuration file for the parallel-dev plugin.
---

# Initialize Parallel-Dev Config

Create a `.claude/parallel-dev.local.md` configuration file for this project.

## Steps

1. Check if `.claude/parallel-dev.local.md` already exists. If so, read it and ask the user if they want to replace or update it.

2. Explore the project to determine which agents are relevant:
   - Presence of `*.java`, `*.kt`, `*.scala` files (non-Android) → `jvm`
   - Presence of `*.ts`, `*.tsx`, `*.js`, `package.json` → `web`
   - Presence of `AndroidManifest.xml` → `android`
   - Presence of `Dockerfile`, `*.tf`, `.github/workflows/` → `infrastructure`
   - Presence of `BUILD`, `WORKSPACE`, `*.bzl` → `build`
   - Presence of migration directories (`db/migrate`, `src/main/resources/db/migration`, etc.) → `data`

3. Determine a sensible `worktree_base`. Default is `.worktrees`. If the project already has a convention (e.g., a specific tmp or scratch directory), use that.

4. Write `.claude/parallel-dev.local.md`:

```markdown
---
worktree_base: .worktrees
active_agents:
  - <only the agents detected as relevant>
---

# Project Notes for Parallel Dev

<Add any project-specific context here that agents should know:>
<- Monorepo structure overview>
<- Framework conventions (e.g., "uses Spring Boot 3 with Kotlin")>
<- API style (REST/gRPC/GraphQL)>
<- Test framework in use>
<- Build tool and version>
```

5. Add `.worktrees/` (or the configured `worktree_base`) to `.gitignore` if not already present.

6. Report to the user:
   - Which agents were enabled and why
   - The worktree base directory
   - What to add to the project notes section
