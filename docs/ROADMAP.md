# Roadmap

1. **Control node up**: LXC, `control_node.yml`, keys into `.sops.yaml` and vars. Repo private.
2. **Discovery**: Proxmox guests + Technitium export → inventory + `docs/INVENTORY.md`.
3. **DNS as code**: export → `dns/<zone>.yml` → plan shows ZERO DIFF.
4. **Baseline rollout**: lab → wb non-critical → cloud → critical. Verify keys, then harden sshd.
5. **Certs**: ACME DNS-01 via Cloudflare. Wildcards per zone on the reverse proxy, Proxmox built-in ACME.
6. **Reverse proxy**: Traefik or Caddy. Pick one, write a role, put services behind it.
7. **Tailscale ACLs as code**: tags `ctrl`/`server`/`agent`, split DNS for the three zones, OAuth client for tagged ephemeral keys.
8. **Services as Compose stacks**: one role per stack, `/opt/stacks/<name>`, data on separate volumes, PBS before changes.
9. **Identity**: Authentik federated with Entra ID; OIDC for Proxmox, Grafana, Semaphore; keep break-glass local admins.
10. **SSH certs (optional)**: step-ca with OIDC via Authentik, so short-lived certs replace static keys.
11. **Visibility**: Fleet (osquery) for drift/compliance queries; Intune only for laptops/phones.
12. **Semaphore UI** on ctrl, behind Authentik, for scheduled baseline runs and run history.
