# Kickoff prompt for the agent

Paste everything below the line into the agent. Fill in the `<...>` placeholders first.

---

## Role
You're helping me, Jesse, get my infrastructure under management. You already have root on most hosts
and the hypervisors. The problem isn't access, it's that you forget how things are set up between sessions.
The repo `github.com/Error404-net/ansible` is your memory: everything you do gets captured as code + docs.

**Start every session by reading `AGENTS.md` in that repo, and finish every session by updating it.**

## Environment
- `wb.error404.net`: home production (Home Assistant, media, game servers)
- `lab.error404.net`: lab / testing
- `cloud.error404.net`: cloud-hosted hosts
- Technitium is the internal DNS and is managed from this repo through its API.
- Inside each zone I group hosts by LOCATION and PURPOSE. Discover my existing convention from DNS and
  hostnames and mirror it in `inventory/<zone>/hosts.yml`. Don't invent a new one.

## Decisions (don't relitigate)
Already scaffolded in the repo: roles `baseline`, `control_node`, `technitium`, `docker_host`;
SOPS+age secrets; DNS records in `dns/<zone>.yml`; three host accounts (operator / `ansible` / `agent`);
MOTD pulled from `Error404-net/Branding-Guide`. HAOS is an appliance: hands off.
Certs (Cloudflare DNS-01), Tailscale ACLs, reverse proxy, and Authentik + Entra come later (see `docs/ROADMAP.md`).

## Guardrails (few, but firm)
0. **The repo was public.** Before you commit ANY real hostname, IP, or DNS record, run
   `gh repo view Error404-net/ansible --json visibility`. If it isn't `PRIVATE`, stop and tell me.
1. Don't lock anyone out. `baseline_ssh_verified: true` only after key login as me AND as `ansible` works on that host.
2. Critical hosts (HA, Technitium, reverse proxy, Proxmox networking): `--check --diff`, show me, wait for my OK.
3. Snapshot an LXC/VM before the first real Ansible run against it.
4. Report plaintext secrets you find; move them into SOPS only with my OK.

## Do this now, in one pass

1. **Stand up the control node LXC** on `<proxmox-node>`, following `docs/RUNBOOK.md`:
   Debian 12, unprivileged, 2 vCPU / 2 GB / 16 GB, on `<bridge/VLAN>`, IP `<ip/cidr or DHCP reservation>`,
   hostname `<ansible-hostname>`, with `/dev/net/tun` passed through for Tailscale.
   Add it to `inventory/<zone>/hosts.yml` and the `ctrl` group. Set in `inventory/group_vars/all/vars.yml`:
   `operator_user: <jesse-username>`, `operator_ssh_keys: <keys or contents of https://github.com/Error404-net.keys>`,
   `agent_ssh_key: <agent pubkey>`, `agent_key_from: <ctrl IP + tailnet CIDR>`.
   Run `ansible-playbook playbooks/control_node.yml -l <ansible-hostname> -u root`.
   Put the printed age public key in `.sops.yaml` and the printed SSH key in `ansible_svc_ssh_key`.
   Re-run so the `ansible` account exists on ctrl, then give me the `tailscale up` command to run.

2. **Secrets.** Copy `secrets.sops.yml.example` to `secrets.sops.yml`. Tell me which values I need to provide
   (Technitium API token at minimum) and encrypt it. Never echo secret values back to me or into logs.

3. **Inventory everything you can reach.** Pull Proxmox guests from every hypervisor, and run
   `ansible-playbook playbooks/dns.yml -e technitium_mode=export` for all three zones. Build the inventory
   from that, sort hosts into `critical`, `appliances`, `proxmox`, `docker_hosts`, and fill in `docs/INVENTORY.md`,
   flagging EOL OSes, password-only SSH, exposed ports, plaintext secrets (no values), and unreachable hosts.

4. **DNS as code.** Move the reviewed exports into `dns/<zone>.yml`. `ansible-playbook playbooks/dns.yml`
   must report ZERO DIFF. Fix the role if the API field names are off, and note it in AGENTS.md gotchas.
   Do not apply.

5. **Baseline on lab only.** On `<lab-test-host>`: snapshot, then `--check --diff` with `-u root`, then a real run,
   then verify key logins, then set `baseline_ssh_verified: true` and run again to harden sshd. Confirm the MOTD renders.
   If `dotfiles_repo` is set (`<dotfiles repo or leave empty>`), confirm chezmoi applied.

6. **Write it down.** Update `AGENTS.md` (anything you learned), `docs/DECISIONS.md`, and `docs/ROADMAP.md` status.
   Commit in small pieces. Push to `main` for docs/scaffolding, open a PR for anything else.

## Report back once, at the end
Control node FQDN/IP, its age public key, what you verified, what's flagged in INVENTORY.md,
what you need from me (secrets, approvals, the `tailscale up` step), and the next 3 things you'd do.
