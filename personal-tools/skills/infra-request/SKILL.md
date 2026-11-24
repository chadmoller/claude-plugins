---
name: infra-request
description: File a request for a homelab infrastructure change (DNS name, K8s namespace/ingress, secret, database, network rule, runner capability, etc.) to the mollermalt-infra repo on Gitea. Use when work in another project needs something from the infra that the project can't or shouldn't change itself, or when the user says "file an infra request" / "ask infra for X".
---

# File an infra request

Infra changes for the homelab (`chad/mollermalt-infra` on `git.taproom.mollermalt.com`)
are **not** made from other projects. File a request issue instead. The user clears
requests by hand in a mollermalt-infra session. Nothing processes them automatically,
so don't wait on one. Say it's filed and keep going, or stop if the work is blocked.

All commands use the `tea` CLI with the `taproom` login. Always pass
`--login taproom --repo chad/mollermalt-infra` explicitly.

## 1. Check for an existing request

```bash
tea issues list --login taproom --repo chad/mollermalt-infra --labels request --state open \
  --fields index,title,labels,url
```

If one already covers the need, add context with a comment instead of filing a duplicate:

```bash
tea comment --login taproom --repo chad/mollermalt-infra <index> "<extra context>"
```

## 2. Make sure the source label exists

The source label is `from/<project>`, where `<project>` is the current repo name (e.g. `from/cwest`).

```bash
tea labels list --login taproom --repo chad/mollermalt-infra -o simple | grep -q "from/<project>" || \
  tea labels create --login taproom --repo chad/mollermalt-infra \
    --name "from/<project>" --color "#5319e7" --description "Requested by chad/<project>"
```

## 3. File the request

Write the body to a temp file first. Keep it self-contained, because the infra session
won't have this project's context. Include:

- **Requesting project:** `chad/<project>`, plus the branch or commit if relevant
- **What is needed:** the concrete change. Name hostnames, ports, namespaces, sizes.
  Describe the *need*, not the infra implementation. The infra repo has its own
  conventions, e.g. purpose-based DNS names under `taproom.mollermalt.com`.
- **Why / context:** what this project is doing, with links to specs or design docs
- **Done when:** a concrete check the requester can run
- **Urgency:** Blocking / Soon / Whenever

```bash
tea issues create --login taproom --repo chad/mollermalt-infra \
  --title "[request] <short summary>" \
  --labels "request,from/<project>" \
  --description "$(cat <body-file>)"
```

Report the issue URL to the user. If this project tracks pending work (a TODO, an
OpenSpec task, etc.), note there that it waits on that infra request URL.

## Checking status later

```bash
tea issues list --login taproom --repo chad/mollermalt-infra --labels "from/<project>" --state all \
  --fields index,title,state,labels,url
tea issues --login taproom --repo chad/mollermalt-infra <index> --comments
```

A request with the `needs-info` label has a question waiting in its comments. Answer it
with `tea comment`. When the request is closed, the closing comment says what changed and
how to use it.

## If tea isn't logged in

`tea logins list` should show `taproom`. If it doesn't, ask the user to set it up. The steps are in
mollermalt-infra `kubernetes/apps/gitea/manual-config.md`, "Phase 6 — Infra request pipeline".
Never create tokens yourself.
