# sensible-ops

Ansible playbook repo for managing personal servers. Enforces baseline system state across all managed hosts.

## Servers

| Host | Group | Notes |
|------|-------|-------|
| sudostorage.sudoservers.com | local | Tailscale, ZFS, extra packages |
| brazil.tail1d0df.ts.net | remote | |

## Roles

| Role | Description |
|------|-------------|
| `base` | Hostname, timezone, common packages |
| `ufw` | Disables UFW (firewall managed elsewhere) |
| `users` | Creates `will` user, groups, SSH key, sudo, dotfiles via yadm |
| `docker` | Installs Docker + Compose, configures daemon |
| `tailscale` | Adds Tailscale repo, installs and starts tailscaled (sudostorage only) |

## Requirements

- Ansible installed on the control machine
- SSH access to managed hosts as `will`
- Vault password at `~/.ansible/vault_pass`
- Host keys in `~/.ssh/known_hosts` (run `ssh-keyscan <host> >> ~/.ssh/known_hosts` for new hosts)

## Usage

```bash
# Run against all servers
ansible-playbook playbooks/site.yml

# Run against one group
ansible-playbook playbooks/site.yml --limit local
ansible-playbook playbooks/site.yml --limit remote

# Run a single role across all servers
ansible-playbook playbooks/site.yml --tags docker
ansible-playbook playbooks/site.yml --tags users

# Dry run
ansible-playbook playbooks/site.yml --check --diff
```

## Vault

Secrets are encrypted with `ansible-vault` and stored in `inventory/group_vars/all/vault.yml`.

```bash
# Edit secrets
ansible-vault edit inventory/group_vars/all/vault.yml

# Encrypt after creating a new vault file
ansible-vault encrypt <file>
```

The vault password lives at `~/.ansible/vault_pass` (never committed).

## Adding a New Host

1. Add the host to `inventory/hosts.yml` under the appropriate group (`local` or `remote`)
2. Run `ssh-keyscan <host> >> ~/.ssh/known_hosts`
3. Create `host_vars/<hostname>/vars.yml` if host-specific overrides are needed
