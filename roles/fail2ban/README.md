# ctsc.sys.fail2ban

Installs `fail2ban` (apt) and ensures the service is running and enabled, using
the package's own default jail configuration.

No variables. This role deliberately doesn't manage `jail.local` /
`jail.d/*.conf` — that's inherently host- or fleet-specific (which jails,
which `ignoreip` ranges). If a consuming inventory needs custom jails, drop a
`template`/`copy` task after this role in the calling playbook, or extend
this role with a `jail_config` variable later if the need turns out to be
common across inventories.

## Example

```yaml
- hosts: all
  become: true
  roles:
    - ctsc.sys.fail2ban
```
