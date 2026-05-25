---
name: parallel-dev:parallel-dev
version: 1.0.0
description: This skill should be used when the user wants to "run parallel-dev", "use parallel-dev", "develop in parallel", "run a multi-agent task", "parallelize this feature", or wants to execute a development task across multiple domains (JVM, web, Android, infrastructure, build, data) concurrently using isolated git worktrees. After merging all domain work, automatically pushes the feature branch, opens a PR, monitors CI, fixes failures, and merges to main on success.
---

# Run Parallel Dev

Invoke the `/parallel-dev` command with the user's task description as the argument.
