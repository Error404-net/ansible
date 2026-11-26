# Decisions

One line each, newest at the bottom. `YYYY-MM-DD: decision (why)`

- 2026-11-26: Repurposed Error404-net/ansible as the infra source of truth; old stub playbooks removed (stale keys, unused since 2024).
- 2026-11-26: Ansible is the config backbone; Fleet/Intune later and only for visibility/client endpoints.
- 2026-11-26: Disposable control node LXC with a shared /opt/infra checkout for Jesse + agent; repo is the source of truth, not the host.
- 2026-11-26: Secrets via SOPS+age (readable diffs, multiple recipients) over ansible-vault.
- 2026-11-26: Zones wb/lab/cloud.error404.net; groups zone → location → purpose, mirroring existing DNS naming.
- 2026-11-26: Technitium managed via its API from this repo; role never deletes records.
- 2026-11-26: Three host accounts: operator (Jesse), `ansible` service (key only on ctrl), `agent` (limited sudo, one-line revoke).
- 2026-11-26: MOTD script pulled from Branding-Guide at a pinned ref instead of copied.
- 2026-11-26: No fonts on servers; fonts belong on client machines.
