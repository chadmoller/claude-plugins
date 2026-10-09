---
name: tofu-apply
version: 1.0.0
description: This skill should be used when the user asks to "apply", "run tofu", "tofu apply", "apply the terraform", "deploy network changes", or wants to run OpenTofu against the opnsense or unifi network layers in this repo. Handles sops credential decryption automatically.
---

# Tofu Apply

Run OpenTofu plan/apply for the network layers in this repo (`network/opnsense` or `network/unifi`). Handles sops credential decryption automatically.

## Credential pattern

Both layers store encrypted credentials in a `secrets.sops.env` file one level above the `terraform/` directory. Decrypt and export before running tofu:

```bash
set -a && eval "$(sops --decrypt --output-type dotenv ../secrets.sops.env)" && set +a
```

## Paths

| Layer | Terraform dir | Secrets file |
|---|---|---|
| OPNsense | `network/opnsense/terraform/` | `network/opnsense/secrets.sops.env` |
| UniFi | `network/unifi/terraform/` | `network/unifi/secrets.sops.env` |

## Workflow

1. **Determine the target layer** from context (e.g. recent edits, user's explicit mention).
   If ambiguous, ask: "Should I apply opnsense, unifi, or both?"

2. **Determine the command** — default to `apply`. Use `plan` if the user asked to preview
   or if there's any uncertainty about what will change.

3. For each target layer, run from the repo root:

   ```bash
   cd network/<layer>/terraform
   set -a && eval "$(sops --decrypt --output-type dotenv ../secrets.sops.env)" && set +a && tofu <command>
   ```

   Use `-auto-approve` only when the user explicitly said "apply" (not just "plan").

4. **Report** what was created/changed/destroyed. If the plan showed unexpected changes,
   stop and ask before proceeding to apply.

## Error handling

- If sops decryption fails (e.g. missing age key), tell the user and stop — do not proceed.
- If tofu plan shows changes the user didn't ask for, surface them and ask before applying.
- If tofu apply fails mid-way, show the error and do not retry automatically.
