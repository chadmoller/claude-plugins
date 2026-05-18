---
name: plan-branch
description: This skill should be used when the user asks to "plan a branch", "create a plan and branch", "plan this feature", "draft a plan and commit it", or wants to create a development plan that automatically becomes a git branch with the plan committed into the repo.
---

# Plan Branch

Create a development plan, then automatically create a git branch named after the plan and commit the plan file into the repository on that branch.

## Workflow

### Phase 1: Plan Creation

1. Call EnterPlanMode to enter plan mode
2. Explore the codebase as needed to understand the task
3. Write the plan to the plan file path provided by the plan mode system
4. The first line of the plan file MUST be a concise title heading:
   `# <Short Title>` (3–6 words, suitable for a git branch name)
   Example: `# Add User Authentication`
5. Call ExitPlanMode to present the plan for user approval and wait for approval

### Phase 2: Branch and Commit (runs after plan approval)

1. **Read the plan file** (path was provided in the plan mode system message)
   and extract the title: the text of the first `# ` heading on line 1

2. **Derive the branch name** from the title:
   - Strip the leading `# `
   - Lowercase everything
   - Replace spaces and non-alphanumeric characters with hyphens `-`
   - Collapse consecutive hyphens into one
   - Strip leading/trailing hyphens
   - Truncate to 50 characters
   - Example: `Add User Authentication` → `add-user-authentication`

3. **Check for branch conflicts**: run `git branch --list <branch-name>`.
   If the branch already exists, append `-2`, `-3`, etc. until unique.

4. **Create the git branch**: `git checkout -b <branch-name>`

5. **Copy the plan into the repo**:
   - Ensure a `plans/` directory exists at the repo root (`mkdir -p plans`)
   - Get the current UTC time in ISO-8601 format: `date -u +"%Y-%m-%dT%H:%M:%SZ"`
   - The destination filename is `<timestamp>-<branch-name>.md`
     (colons in the timestamp replaced with hyphens to be filesystem-safe,
     e.g. `2026-05-17T14-32-00Z-add-user-authentication.md`)
   - Copy the plan file: `cp <plan-file-path> "plans/<timestamp>-<branch-name>.md"`

6. **Commit**:
   ```
   git add "plans/<timestamp>-<branch-name>.md"
   git commit -m "Add plan: <original title text>"
   ```

7. **Report** to the user: branch name, plan file path in repo, and next steps.

## Error Handling

- If not inside a git repository when Phase 2 runs, report the error clearly
  and tell the user they can manually run the git steps.
- If the plan file destination already exists in the repo, overwrite it.
