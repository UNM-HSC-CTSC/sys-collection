# ctsc.sys.xrdp

Installs Xfce (`xfce4`, `xfce4-goodies`) and `xrdp` for remote desktop
access, and works around xrdp's usual pitfalls on Ubuntu:

- Adds the `xrdp` user to the `ssl-cert` group, which it needs to read the
  self-signed TLS cert xrdp generates for the RDP connection.
- Sets Xfce as the system-wide default session (via the `x-session-manager`
  update-alternatives group), since Ubuntu also ships GNOME and xrdp's
  session negotiation with GNOME is broken -- logging in over RDP with the
  GNOME default typically produces a black screen or an immediate
  disconnect. This is system-wide rather than per-user, so it also covers
  users created after this role runs.
- Sets `MaxSessions` in `/etc/xrdp/xrdp.ini` (default `5`; xrdp itself
  defaults to `50`).

## Variables

| Variable            | Default | Description                                                    |
| ------------------- | ------- | -------------------------------------------------------------- |
| `xrdp_max_sessions` | `5`     | Maximum concurrent RDP sessions (`MaxSessions` in `xrdp.ini`). |

## Example

```yaml
- hosts: all
  become: true
  roles:
    - role: ctsc.sys.xrdp
      vars:
        xrdp_max_sessions: 10
```
