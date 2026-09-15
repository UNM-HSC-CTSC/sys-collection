# ctsc.sys.ufw

Sets `ufw` to a `deny incoming` / `allow outgoing` baseline, applies an
allowlist from `ufw_allow_rules`, then enables it.

The role is fully idempotent: it diffs the currently active `allow`/`limit`
rules against `ufw_allow_rules` and only touches what's actually different,
so a repeat run with unchanged variables reports no changes. The end state
is still declarative and not additive — any `allow`/`limit` rule not
present in `ufw_allow_rules`, including ones added outside of Ansible (e.g.
via a manual `ufw allow ...` or `ufw limit ...`), is removed. This pruning
is scoped to rules managed by this role: `deny`/`reject` rules added
outside the role are left alone.

## Variables

| Variable          | Default | Description                                                                                                                    |
| ----------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `ufw_allow_rules` | `[]`    | List of `{port, proto, src, rule, comment}` dicts. `port` is required (a port number/name/range, or `"any"`); `proto`/`src`/`rule`/`comment` are optional (default `any`/`any`/`allow`/`''`). |

`port: "any"` allows all ports -- typically paired with an explicit `src` to
scope it, such as trusting an entire subnet like a Docker bridge network.

`rule` selects the ufw action: `"allow"` (the default) permits the
connection outright, while `"limit"` rate-limits repeated connection
attempts from the same source (ufw denies an IP that attempts to connect 6
or more times within a 30 second period). This is useful for
internet-facing services like SSH where you want to allow access but guard
against brute-force attempts.

## Example

```yaml
- hosts: all
  become: true
  roles:
    - role: ctsc.sys.ufw
      vars:
        ufw_allow_rules:
          - { port: "ssh", proto: tcp, rule: limit, comment: "SSH" }
          - { port: "443", proto: tcp, comment: "HTTPS" }
          - {
              port: "2049",
              proto: tcp,
              src: "192.168.100.2",
              comment: "NFS from peer",
            }
          - {
              port: "any",
              src: "172.18.0.0/16",
              comment: "Docker monitoring network",
            }
```
