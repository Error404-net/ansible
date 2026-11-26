# AGENTS.md: read this first, every session

You are working on Error404's infrastructure repo. You (the AI agent) have root
on most hosts and hypervisors but **no memory between sessions**. This file and
`docs/` are your memory. Read them before doing anything. Update them before you
finish. If you learned it and didn't write it here, you'll have to learn it again.

## Environment
- Zones (Technitium is authoritative internally):
  - `wb.error404.net`: **home production** (Home Assistant, media, game servers). The household notices outages.
  - `lab.error404.net`: lab/testing. New roles land here first.
  - `cloud.error404.net`: cloud-hosted hosts, reached over Tailscale.
- Hosts are grouped **zone → location → purpose**, mirroring the existing naming in DNS.
  Cross-cutting groups live in `inventory/zz-functional.yml`: `critical`, `appliances`, `proxmox`, `docker_hosts`, `ctrl`.
- Control node: the `ctrl` group. Shared checkout at `/opt/infra` (group `infra`), used by Jesse and you.
  Tools in `/opt/infra-venv/bin` (on PATH via `/etc/profile.d/infra.sh`).
- Ansible connects as `ansible` with `/etc/infra/ssh/id_ed25519`. First run on a new host: `-u root`.
- Secrets: SOPS+age. `SOPS_AGE_KEY_FILE=/etc/infra/age/keys.txt`. Edit with `sops <file>`.
- DNS as code: `dns/<zone>.yml` + `playbooks/dns.yml` (export | plan | apply).
- Brand MOTD comes from `Error404-net/Branding-Guide` at `branding_ref`. Don't fork it here.

## Guardrails
1. **Repo visibility:** before committing real hostnames/IPs, confirm with
   `gh repo view Error404-net/ansible --json visibility`. If it's PUBLIC, stop and tell Jesse.
2. **No lockouts:** `baseline` hardens sshd only when `baseline_ssh_verified: true` is set in host_vars.
   Set it only after a key login as the operator AND as `ansible` actually worked.
   `AllowUsers` + `PermitRootLogin no` will cut off root-key access, so make sure `ansible_svc_ssh_key` is set first.
3. **Critical hosts** (HA, Technitium, reverse proxy, Proxmox networking): `--check --diff`, show Jesse, wait for OK.
4. **Snapshot** an LXC/VM before the first real run against it.
5. **Secrets:** never in plaintext. Report any you find; move them into SOPS only with Jesse's OK.
6. **Appliances** (HAOS): never run baseline against them.

## Workflow
- Small commits, clear messages. Push straight to `main` for docs and scaffolding.
  Open a PR for sshd/sudoers/firewall/DNS-record changes or anything touching `critical`.
- Before a real run: `ansible-lint`, `yamllint .`, `--check --diff`.
- Order of rollout: lab → wb (non-critical) → cloud → critical.

## Where things are
| Want | Look in |
|---|---|
| What hosts exist / what runs where | `docs/INVENTORY.md`, `inventory/*/hosts.yml` |
| How to rebuild ctrl / break-glass | `docs/RUNBOOK.md` |
| Why something is the way it is | `docs/DECISIONS.md` |
| What's next | `docs/ROADMAP.md` |
| Kickoff instructions | `prompt.md` |

## Gotchas (append as you find them)
- Files in `inventory/` are parsed as inventory. Non-inventory data (DNS records) lives in `dns/`.
- Technitium API field names (`rData.ipAddress`, `rData.cname`) were written from the API docs, not tested live. Verify on first export.
- Debian's stock `sshd_config` includes `sshd_config.d/*.conf` at the top; first value wins, so the drop-in overrides.
- Offline tests (2026-11-26, mock Technitium on 127.0.0.1, RFC 5737 data): export, plan (ZERO DIFF and drift), and apply with `technitium_confirm_apply=true` work with the field names above. Apply POSTs `/api/zones/records/add` with `overwrite=true`. Still unverified against a real Technitium.
- Open risk: Technitium `overwrite=true` on a name may replace the whole RRset, not just one value. That would break "never deletes" for multi-IP names. Verify on live Technitium before any apply that touches multi-value names.
- `baseline` fails under `-i localhost,` on undefined `timezone`, `agent_ssh_key`, `dotfiles_repo` (they come from group_vars). Define defaults in `roles/baseline/defaults/main.yml`.
- Test lockout guards with a real list: `-e '{"operator_ssh_keys": []}'`. `-e key=[]` passes the string "[]".
- `gh` was unauthenticated (401) on the test host, so the repo-visibility guardrail is unchecked.
