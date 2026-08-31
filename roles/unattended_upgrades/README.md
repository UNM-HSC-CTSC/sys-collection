# ctsc.sys.unattended_upgrades

Installs `unattended-upgrades`, renders `/etc/apt/apt.conf.d/50unattended-upgrades`
from a package-blacklist variable, and (optionally) turns on the periodic
`apt-daily`/`apt-daily-upgrade` timers via `/etc/apt/apt.conf.d/20auto-upgrades`.

## Variables

| Variable | Default | Description |
|----------|---------|--------------|
| `unattended_upgrades_package_blacklist_content` | `""` | Raw block of Python-regex package-name patterns (one `"pattern";` per line) rendered into `Unattended-Upgrade::Package-Blacklist`. |
| `unattended_upgrades_enable_periodic` | `true` | Whether to write `20auto-upgrades` and enable the `apt-daily*` timers. Set `false` if the inventory manages periodic APT runs itself. |

## Example

```yaml
- hosts: all
  become: true
  roles:
    - role: ctsc.sys.unattended_upgrades
      vars:
        unattended_upgrades_package_blacklist_content: |
          // Prevent unattended postgres upgrades -- restarts trash running queries
          "postgresql-.*";
          "libpq.*";
```
