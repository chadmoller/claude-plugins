---
description: Run a development task across multiple domains in parallel using isolated git worktrees. Each domain (JVM, web, Android, infrastructure, build, data) runs concurrently and results are merged sequentially.
argument-hint: <task description>
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, TaskCreate, TaskGet, TaskList, TaskStop, Agent, EnterPlanMode, ExitPlanMode, EnterWorktree, ExitWorktree]
---

# Parallel Dev

Your task: $ARGUMENTS

Execute the parallel-dev multi-agent workflow. Follow every phase below in order. Do not skip any phase.

---

## Phase 0: Config & Preflight

1. Read `.claude/parallel-dev.local.md` if it exists. Extract from YAML frontmatter:
   - `worktree_base`: directory for worktrees (default: `.worktrees`)
   - `active_agents`: list of enabled domain agents (default: all — jvm, web, android, infrastructure, build, data)
   - Any project-specific notes in the body to pass to agents

   If the file does not exist, ask the user if they want to create one now. Offer a template with the full list of agents and default worktree path. Do not proceed until the config exists or the user confirms defaults.

2. Verify the repository is clean: `git status --porcelain`. If there are uncommitted changes, warn the user and stop.

3. Ensure `<worktree_base>` is in `.gitignore` (append if missing).

---

## Phase 1: Planning

Invoke the **planner** agent using the Agent tool with:
- The full task description
- The project root path (`pwd`)
- The list of active agents
- Project notes from config (if any)

The planner returns two sections:
- **PLAN DOCUMENT**: A human-readable Markdown plan with a `# Title` on the first line
- **WORK BREAKDOWN**: Per-domain task assignments and a suggested merge order

Parse both sections. Filter the work breakdown to only include domains in `active_agents`.

If no domains have work, report this to the user and stop.

---

## Phase 2: Plan Approval & Branch Creation

**This phase is mandatory. Do not skip it.**

### 2a. Present the plan for approval

1. Call `EnterPlanMode` to enter plan mode. Note the plan file path provided by the system.
2. Write the plan document content (from the planner) to the plan file path.
   - The first line must be `# <Short Title>` — preserve it exactly as the planner wrote it.
3. Call `ExitPlanMode` to present the plan to the user and wait for approval.
4. **Do not proceed until the user approves the plan.**
   If the user declines or requests changes, stop or revise accordingly.

### 2b. Create the feature branch

After user approval:

1. Read the plan file (path provided in the plan mode system message). Extract the title from the first `# ` heading.

2. Derive the branch name from the title:
   - Strip the leading `# `
   - Lowercase everything
   - Replace spaces and non-alphanumeric characters with hyphens `-`
   - Collapse consecutive hyphens into one
   - Strip leading/trailing hyphens
   - Truncate to 50 characters

3. Check for branch conflicts: `git branch --list <branch-name>`.
   If it already exists, append `-2`, `-3`, etc. until unique.

4. Create and switch to the feature branch: `git checkout -b <branch-name>`

5. Copy the plan into the repo:
   - `mkdir -p plans`
   - Get UTC timestamp: `date -u +"%Y-%m-%dT%H:%M:%SZ"` (replace colons with hyphens for the filename)
   - `cp <plan-file-path> "plans/<timestamp>-<branch-name>.md"`

6. Commit:
   ```
   git add "plans/<timestamp>-<branch-name>.md"
   git commit -m "Add plan: <original title text>"
   ```

Record the feature branch name — you will pass it to the orchestrator.

---

## Phase 3–7: Parallel Execution, Merge & Review

Invoke the **orchestrator** agent using the Agent tool, passing:
- The full task description
- The project root path (absolute)
- The feature branch name (created in Phase 2b)
- The work breakdown (domain task assignments and merge order from the planner)
- The `worktree_base` path
- Project notes from config (if any)
- The list of active agents

The orchestrator will spawn domain workers in parallel, merge results sequentially, run a code review, and produce a final report.
