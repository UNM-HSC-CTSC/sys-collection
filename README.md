# Ansible Collection - ctsc.sys

Host-agnostic Ansible roles for use with CTSC infrastructure (servers, VMs, etc).

## Roles

| Role                                               | Description                                                   |
| -------------------------------------------------- | ------------------------------------------------------------- |
| [`conda`](roles/conda)                             | Install a system-wide Miniconda for a multi-user environment  |
| [`docker`](roles/docker)                           | Install Docker Engine and (optionally) fix ufw for containers |
| [`fail2ban`](roles/fail2ban)                       | Install and enable fail2ban with distro defaults              |
| [`ufw`](roles/ufw)                                 | Install UFW and apply an allowlist                            |
| [`unattended_upgrades`](roles/unattended_upgrades) | Configure unattended-upgrades and periodic APT timers         |
| [`xrdp`](roles/xrdp)                               | Install Xfce and xrdp for remote desktop access                |

## Using this collection from another repo

```yaml
# requirements.yml
collections:
  - name: https://github.com/UNM-HSC-CTSC/sys-collection.git
    type: git
    version: main
```

```shell
ansible-galaxy collection install -r requirements.yml
# or if already installed
ansible-galaxy collection install -r requirements.yml --force
```

```yaml
# a playbook
- hosts: all
  become: true
  roles:
    - role: ctsc.sys.fail2ban
    - role: ctsc.sys.ufw
      vars:
        ufw_allow_rules: "{{ my_hosts_allow_rules }}"
    - role: ctsc.sys.unattended_upgrades
      vars:
        unattended_upgrades_package_blacklist_content: "{{ my_hosts_blacklist }}"
```

## Development

```shell
uv venv && source .venv/bin/activate
uv pip sync requirements.txt
```

(any other `requirements.txt`-compatible tool works too, e.g. `pip install -r requirements.txt`)

Then, from the repo root:

```shell
molecule test -s conda
molecule test -s docker
molecule test -s fail2ban
molecule test -s ufw
molecule test -s unattended_upgrades
molecule test -s xrdp
ansible-lint
yamllint .
```

Each role gets its own scenario under `extensions/molecule/<role_name>/`.
Scenarios share a single base image, `extensions/molecule/Dockerfile`, which
each scenario's `molecule.yml` points to via `dockerfile: ../Dockerfile` —
add packages there only if a new scenario actually needs them.
