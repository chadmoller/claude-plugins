---
description: Run a development task across multiple domains in parallel using isolated git worktrees. Each domain (JVM, web, Android, infrastructure, build, data) runs concurrently and results are merged sequentially.
argument-hint: <task description>
allowed-tools: [Read, Write, Edit, Bash, Glob, Grep, TaskCreate, TaskGet, TaskList, TaskStop, Agent, EnterWorktree, ExitWorktree]
---

# Parallel Dev

Invoke the **orchestrator** agent to execute the following task using the parallel-dev multi-agent workflow:

$ARGUMENTS

The orchestrator will:
1. Read `.claude/parallel-dev.local.md` for project configuration
2. Run the **planner** to analyze the task and codebase
3. Spawn domain agents in parallel, each in their own git worktree
4. Sequentially merge all results back to the current branch
5. Run a final **code review**

If `.claude/parallel-dev.local.md` does not exist in this project, ask the user if they want to create one now. Offer a template with the full list of agents and default worktree path.
