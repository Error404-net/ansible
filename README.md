```
================================================================================

  █████╗ ███╗   ██╗███████╗██╗██████╗ ██╗     ███████╗
 ██╔══██╗████╗  ██║██╔════╝██║██╔══██╗██║     ██╔════╝
 ███████║██╔██╗ ██║███████╗██║██████╔╝██║     █████╗
 ██╔══██║██║╚██╗██║╚════██║██║██╔══██╗██║     ██╔══╝
 ██║  ██║██║ ╚████║███████║██║██████╔╝███████╗███████╗
 ╚═╝  ╚═╝╚═╝  ╚═══╝╚══════╝╚═╝╚═════╝ ╚══════╝╚══════╝

                 -=[ infrastructure as code, finally ]=-
                            by Error404

================================================================================

  RELEASE.....: ansible (repurposed, v2)
  DATE........: 2026
  STATUS......: scaffold. control node first, everything else after
  STACK.......: ansible-core / sops+age / technitium API / proxmox LXC
  ZONES.......: wb.error404.net    (home production)
                lab.error404.net   (lab / testing)
                cloud.error404.net (cloud hosted)
  TAGS........: homeops / ansible / gitops / ascii / new-wave-hacker
  LICENSE.....: MIT

--------------------------------------------------------------------------------
  WHAT IS THIS
--------------------------------------------------------------------------------

  The sprawl, wrangled. One repo that knows every host, every DNS record,
  every key, and how to rebuild all of it from a single LXC.

  Not a lab. wb.* runs Home Assistant, media, and game servers. If it
  breaks, the house notices. Treat it like production, because it is.

  Also doubles as the AI agent's long-term memory. The agent has root and
  forgets everything between sessions. AGENTS.md is how it remembers.

--------------------------------------------------------------------------------
  HOW IT HANGS TOGETHER
--------------------------------------------------------------------------------

      [ GitHub: Error404-net/ansible ]   <- source of truth (PRIVATE)
                    |
                    v
      [ ctrl LXC  /opt/infra ]           <- Jesse + agent both work here
        ansible / sops / age / tailscale
                    |
        +-----------+-----------+------------------+
        v           v           v                  v
     wb.*        lab.*       cloud.*        Technitium API
   (home prod)  (test first) (via tailnet)   (records as code)

  > hosts grouped by  : zone -> location -> purpose
  > cross-cutting     : critical / appliances / proxmox / docker_hosts / ctrl
  > secrets           : SOPS + age, encrypted in-repo, keys never in git
  > ansible connects  : as `ansible` svc user, key lives only on ctrl
  > agent             : own `agent` user, limited sudo, one-line revoke
  > MOTD              : pulled from Error404-net/Branding-Guide, not copied

--------------------------------------------------------------------------------
  STRUCTURE
--------------------------------------------------------------------------------

    ansible/
      AGENTS.md            read this first. agents AND future-you
      prompt.md            kickoff prompt for the AI agent
      ansible.cfg          inventory/, roles/, sops vars plugin
      requirements.yml     pinned galaxy collections
      .sops.yaml           age recipients (jesse + ctrl)
      inventory/
        wb/hosts.yml       home production
        lab/hosts.yml      lab / testing
        cloud/hosts.yml    cloud hosted
        zz-functional.yml  critical / appliances / proxmox / docker_hosts / ctrl
        group_vars/all/    vars.yml + secrets.sops.yml (encrypted)
        host_vars/         per-host overrides (baseline_ssh_verified lives here)
      dns/                 <zone>.yml record sets for Technitium
      roles/
        baseline/          users, keys, sshd, time, updates, node_exporter, MOTD, dotfiles
        control_node/      venv tooling, sops/age, shared /opt/infra, tailscale
        technitium/        export / plan / apply DNS via API (never deletes)
        docker_host/       docker engine + compose, /opt/stacks
      playbooks/
        ping.yml           are you there
        control_node.yml   build/rebuild the ctrl LXC
        baseline.yml       the everyday one. safe to schedule (minus critical)
        dns.yml            technitium export / plan / apply
        docker_hosts.yml   docker engine
        site.yml           all of it. check first. never scheduled
      docs/
        INVENTORY.md       every host, what it runs, how it's managed
        RUNBOOK.md         rebuild ctrl, break-glass, restore
        DECISIONS.md       dated one-liners. why things are the way they are
        ROADMAP.md         what's next

--------------------------------------------------------------------------------
  QUICK START (on the ctrl LXC)
--------------------------------------------------------------------------------

  $ infra                                   # alias -> cd /opt/infra
  $ ansible-galaxy collection install -r requirements.yml -p collections/
  $ pre-commit install
  $ ansible-playbook playbooks/ping.yml
  $ ansible-playbook playbooks/baseline.yml -l lab --check --diff
  $ ansible-playbook playbooks/dns.yml                    # plan, read-only

  Secrets:

  $ sops inventory/group_vars/all/secrets.sops.yml        # edit in place

  First run against a host that has never seen this repo:

  $ ansible-playbook playbooks/baseline.yml -l <host> -u root --check --diff
  $ ansible-playbook playbooks/baseline.yml -l <host> -u root
    ... verify `ssh <you>@<host>` with a key, then in host_vars/<host>.yml:
        baseline_ssh_verified: true
  $ ansible-playbook playbooks/baseline.yml -l <host>     # now hardens sshd

--------------------------------------------------------------------------------
  RULES OF THE HOUSE
--------------------------------------------------------------------------------

  ⚠ repo MUST be private before real hosts/IPs get committed
  ⚠ no plaintext secrets. *.sops.yml or it doesn't go in. CI checks.
  ⚠ sshd only hardens after key login is verified per host. no lockouts.
  ⚠ critical hosts: --check --diff, then an OK from Jesse. never on a timer.
  ⚠ appliances (HAOS) are hands-off. manage DNS/proxy/certs around them.
  ⚠ snapshot the LXC/VM before the first real run. PBS exists for a reason.
  ✓ lab first. then wb. then cloud. then critical. always that order.
  ✓ the technitium role adds/overwrites. it does not delete. on purpose.

--------------------------------------------------------------------------------
  KNOWN GAPS (this is a scaffold, not a miracle)
--------------------------------------------------------------------------------

  ⚠ inventory is empty until the agent's discovery pass
  ⚠ .sops.yaml has placeholder recipients. fill them or sops refuses
  ⚠ technitium plan only compares A / AAAA / CNAME for now
  ⚠ no reverse proxy, certs, authentik, or tailscale ACLs yet. see ROADMAP
  ✓ the old ssh-setup.yaml is gone. its keys were stale anyway

--------------------------------------------------------------------------------
  LINKS / REFS
--------------------------------------------------------------------------------

  > brand        : https://branding.error404.net
  > brand repo   : https://github.com/Error404-net/Branding-Guide (MOTD source)
  > technitium   : https://github.com/TechnitiumSoftware/DnsServer/blob/master/APIDOCS.md
  > sops         : https://github.com/getsops/sops

================================================================================
         ERROR404.NET  //  !ignore -> return "404: Message not found"
================================================================================
```
