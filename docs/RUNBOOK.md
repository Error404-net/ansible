# Runbook

## Build the control node LXC (from a Proxmox node shell)

```bash
# adjust ID, storage, bridge/VLAN, IP. Debian 12 template.
pveam update && pveam download local debian-12-standard_12.7-1_amd64.tar.zst
pct create <VMID> local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst \
  --hostname <ansible-hostname> --unprivileged 1 --features nesting=1 \
  --cores 2 --memory 2048 --swap 512 --rootfs <storage>:16 \
  --net0 name=eth0,bridge=<bridge>,ip=<ip/cidr>,gw=<gw>,tag=<vlan> \
  --nameserver <technitium-ip> --searchdomain <zone>.error404.net \
  --ssh-public-keys /root/.ssh/authorized_keys --onboot 1
pct start <VMID>
```

Tailscale in an unprivileged LXC needs `/dev/net/tun`. Add to `/etc/pve/lxc/<VMID>.conf`:
```
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

Then from anywhere with this repo:
```bash
ansible-playbook playbooks/control_node.yml -l <ansible-hostname> -u root
```
It prints the ctrl age public key and Ansible SSH public key. Put them in
`.sops.yaml` and `inventory/group_vars/all/vars.yml` (`ansible_svc_ssh_key`).
Then, on ctrl: `tailscale up --advertise-tags=tag:ctrl`.

## Rebuild ctrl from scratch
1. Recreate the LXC (above) and run `control_node.yml`.
2. Restore `/etc/infra/age/keys.txt` from the break-glass copy (password manager), **or**
   generate a new key, add it to `.sops.yaml`, and run `sops updatekeys` on every `*.sops.yml` using Jesse's key.
3. New SSH key? Update `ansible_svc_ssh_key` and run baseline with `-u <operator>`.

## Break-glass
- Proxmox console (`pct enter <VMID>` / VM console) always works. No SSH needed.
- Jesse's age key decrypts every secret without ctrl.
- Revoke the agent everywhere: set `agent_ssh_key: ""`, then run baseline (`exclusive: true` empties its keys).

## Restore
- Guests: Proxmox Backup Server snapshot → restore.
- HAOS: its own backups. Not managed here.
