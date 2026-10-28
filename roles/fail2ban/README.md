# ctsc.sys.fail2ban

Installs `fail2ban` (apt) and ensures the service is running and enabled, using
the package's own default jail configuration.

Optionally sets a global `ignoreip` whitelist in the `[DEFAULT]` section of
`/etc/fail2ban/jail.local`, so it applies to every jail. When the whitelist
changes, the role runs `fail2ban-client reload`, then reads the list back with
`fail2ban-client get sshd ignoreip` and fails if it doesn't match.

The role doesn't manage anything else in `jail.local` / `jail.d/*.conf`.
Which jails to enable is host- or fleet-specific. If a consuming inventory
needs custom jails, drop a `template`/`copy` task after this role in the
calling playbook.

## Variables

| Variable            | Default | Description                                                                                                                                                          |
| ------------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `fail2ban_ignoreip` | `[]`    | IPs / CIDRs / hostnames fail2ban never bans. Empty leaves `jail.local` untouched. Replaces fail2ban's default, so list `127.0.0.1/8` and `::1` too if you want them. |

Notes:

- fail2ban stores CIDRs as their network address (`127.0.0.1/8` becomes
  `127.0.0.0/8`). The read-back check matches plain IPs exactly and checks
  CIDRs only by the number of entries.
- Going back from a non-empty list to `[]` does **not** remove an existing
  `ignoreip` line. Delete it from `jail.local` by hand if you need to.
- The read-back check uses the `sshd` jail, which Ubuntu enables by default.

## Example

```yaml
- hosts: all
  become: true
  roles:
    - role: ctsc.sys.fail2ban
      vars:
        fail2ban_ignoreip:
          - 127.0.0.1/8
          - ::1
          - 192.168.1.1
```
