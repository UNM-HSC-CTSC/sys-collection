# Ansible Collection - ctsc.sys

Host-agnostic Ansible roles for use with CTSC infrastructure (servers, VMs, etc).

## Roles

| Role                                               | Description                                                      |
| -------------------------------------------------- | ---------------------------------------------------------------- |
| [`fail2ban`](roles/fail2ban)                       | Install and enable fail2ban with distro defaults                 |
| [`ufw`](roles/ufw)                                 | Reset ufw to deny-incoming/allow-outgoing and apply an allowlist |
| [`unattended_upgrades`](roles/unattended_upgrades) | Configure unattended-upgrades and periodic APT timers            |

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
molecule test -s fail2ban
ansible-lint
yamllint .
```

Each role gets its own scenario under `extensions/molecule/<role_name>/`.
`fail2ban` and `ufw` have one so far — `unattended_upgrades` needs a scenario
added following the same pattern before it's considered fully ported.
