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
python3 -m venv .venv && source .venv/bin/activate
pip install ansible-core molecule molecule-plugins[docker] ansible-lint yamllint
```

Molecule resolves roles by this collection's FQCN (`ctsc.sys.<role>`), so the
checkout needs to be visible on the collections path under
`ansible_collections/ctsc/sys`. The simplest way is a symlink:

```shell
mkdir -p ~/.ansible/collections/ansible_collections/ctsc
ln -s "$(pwd)" ~/.ansible/collections/ansible_collections/ctsc/sys
```

Then, from the repo root:

```shell
molecule test -s fail2ban   # runs create -> converge -> verify -> destroy
ansible-lint
yamllint .
```

Each role gets its own scenario under `molecule/<role_name>/`. Only
`fail2ban` has one so far — `ufw` and `unattended_upgrades` need scenarios
added following the same pattern before they're considered fully ported.
