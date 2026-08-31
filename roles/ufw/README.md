# ctsc.sys.ufw

Resets `ufw` to a clean `deny incoming` / `allow outgoing` baseline, applies
an allowlist from `ufw_allow_rules`, then enables it.

**Warning**: this role calls `ufw reset` every run. It is idempotent in the
sense that the end state is always exactly `ufw_allow_rules`, but it is not
additive — anything not in the list is removed, including rules added
outside of Ansible.

## Variables

| Variable          | Default | Description                                                                                                   |
| ----------------- | ------- | ------------------------------------------------------------------------------------------------------------- |
| `ufw_allow_rules` | `[]`    | List of `{port, proto, src, comment}` dicts. `proto`/`src`/`comment` are optional (default `any`/`any`/`''`). |

## Example

```yaml
- hosts: all
  become: true
  roles:
    - role: ctsc.sys.ufw
      vars:
        ufw_allow_rules:
          - { port: "ssh", proto: tcp, comment: "SSH" }
          - { port: "443", proto: tcp, comment: "HTTPS" }
          - {
              port: "2049",
              proto: tcp,
              src: "192.168.100.2",
              comment: "NFS from peer",
            }
```
