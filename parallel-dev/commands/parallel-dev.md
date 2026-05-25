---
description: Run a development task across multiple domains in parallel using isolated git worktrees. Each domain (JVM, web, Android, infrastructure, build, data) runs concurrently and results are merged sequentially.
argument-hint: <task description>
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, TaskCreate, TaskGet, TaskList, TaskStop, Agent, AskUserQuestion, EnterPlanMode, ExitPlanMode, EnterWorktree, ExitWorktree, WebFetch, mcp__plugin_github_github__create_pull_request, mcp__plugin_github_github__pull_request_read, mcp__plugin_github_github__merge_pull_request, mcp__plugin_github_github__list_pull_requests]
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

---

## Phase 8: Push, PR, Pipeline & Merge

### 8a. Push the feature branch

Push the feature branch to origin:

```
git push -u origin <branch-name>
```

### 8b. Open a Pull Request

Create a PR using the `mcp__plugin_github_github__create_pull_request` tool:
- **head**: the feature branch name
- **base**: `main`
- **title**: the plan title (from the `# ` heading in the plan file)
- **body**: a brief summary of what was implemented (from the orchestrator's final report)

Display the PR URL to the user.

### 8c. Ask the user whether to manage the PR

Use `AskUserQuestion` to ask:

> "A PR has been opened at <PR URL>. Would you like me to monitor the CI pipeline and automatically merge it on success? (yes/no)"

**If the user says no:**
- Inform them: "The PR is open at <PR URL>. You are responsible for reviewing, managing the pipeline, and merging it when ready."
- Stop here. The workflow is complete.

**If the user says yes:** continue to Phase 8d.

### 8d. Monitor the CI pipeline

Poll the PR status using `mcp__plugin_github_github__pull_request_read` on a loop:
- Wait ~30 seconds between polls: `sleep 30`
- Check the PR's merge status and any reported check/status fields
- Keep the user informed of progress (e.g., "Pipeline running... checks pending")

**Determining check outcomes:**
- If `pull_request_read` returns check run details, use those directly
- If only a URL is available, use `WebFetch` to retrieve the check run page and parse the outcome
- Treat "success", "neutral", or no failing checks as a passing pipeline
- Treat any "failure" or "error" conclusion as a failing pipeline

**If the pipeline fails:**
1. Report the failure to the user with as much detail as available (failing check name, error output)
2. Attempt to fix the failure:
   - Read the relevant source files
   - Apply the minimal fix needed to address the CI error
   - `git add -A && git commit -m "Fix CI: <brief description of fix>"`
   - `git push`
3. Return to the polling loop (Phase 8d) to monitor again

**If the pipeline passes:** continue to Phase 8e.

### 8e. Merge the PR and update local main

1. Merge the PR using `mcp__plugin_github_github__merge_pull_request` with merge method `squash` (or `merge` if the project has no squash preference — use `merge` as the default).
2. Switch to main and pull the merged changes locally:
   ```
   git checkout main
   git pull origin main
   ```
3. Inform the user: "Pipeline passed. PR merged and main is up to date."
