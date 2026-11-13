---
name: check-admin-vlan
version: 1.0.0
description: This skill should be used proactively before running any kubectl command, or when the user asks "am I on Admin VLAN?", "check my network", "can you run kubectl?", or similar. Checks whether the local machine has an IP address in the Admin VLAN (192.168.251.0/24) which is required to reach the Kubernetes API server at 192.168.130.10:6443.
---

# Check Admin VLAN

Before running kubectl (or any command that requires the K8s API server), check whether the
local machine is currently on the Admin VLAN. The K8s API server is only reachable from
192.168.251.0/24 — connections from other VLANs time out silently.

## Steps

1. Run the following command to check for an Admin VLAN address:
   ```bash
   ip addr show | grep -o '192\.168\.251\.[0-9]*'
   ```

2. **If output contains an IP** (e.g. `192.168.251.42`):
   - Admin VLAN is active. Proceed with kubectl commands.
   - You do NOT need to tell the user — just continue working.

3. **If output is empty**:
   - Not on Admin VLAN. Do NOT attempt kubectl — it will hang.
   - Tell the user: "kubectl requires Admin VLAN (192.168.251.0/24) — your current address
     isn't in that range. Please switch networks, then let me know and I'll continue."
   - **Stop and wait** for the user to confirm they've switched before issuing any kubectl commands.

## Notes

- `ip addr` is always safe to run — it's read-only, local, and instant. Use it instead of
  probing kubectl with a timeout.
- If kubectl returns `connection refused` despite an Admin VLAN IP, the API server itself may
  be down — investigate cluster health rather than asking to switch networks.
- Run this check at the start of any task that will need kubectl, not after a failure.
