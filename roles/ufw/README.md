# ctsc.sys.ufw

Sets `ufw` to a `deny incoming` / `allow outgoing` / `deny routed` baseline,
applies an allowlist from `ufw_allow_rules`, then enables it.

The role is fully idempotent: it diffs the currently active `allow`/`limit`
rules against `ufw_allow_rules` (and routed `allow`/`limit` rules against
`ufw_route_rules`) and only touches what's actually different, so a repeat
run with unchanged variables reports no changes. The end state is still
declarative and not additive — any `allow`/`limit` rule not present in
`ufw_allow_rules`, or routed `allow`/`limit` rule not present in
`ufw_route_rules`, including ones added outside of Ansible (e.g. via a
manual `ufw allow ...`/`ufw route allow ...`), is removed. This pruning is
scoped to rules managed by this role: `deny`/`reject` rules added outside
the role are left alone.

## Variables

| Variable             | Default | Description                                                                                                                                                                                   |
| -------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ufw_allow_rules`    | `[]`    | List of `{port, proto, src, rule, comment}` dicts. `port` is required (a port number/name/range, or `"any"`); `proto`/`src`/`rule`/`comment` are optional (default `any`/`any`/`allow`/`''`). |
| `ufw_forward_policy` | `deny`  | Default policy for the FORWARD chain (routed traffic): `"allow"`, `"deny"`, or `"reject"`.                                                                                                    |
| `ufw_route_rules`    | `[]`    | List of `{src, dest, port, proto, rule, comment}` dicts scoping routed traffic to specific subnets. At least one of `src`/`dest` is required; `port`/`proto`/`rule`/`comment` are optional (default `any`/`any`/`allow`/`''`). |

`port: "any"` allows all ports -- typically paired with an explicit `src` to
scope it, such as trusting an entire subnet like a Docker bridge network.

`rule` selects the ufw action: `"allow"` (the default) permits the
connection outright, while `"limit"` rate-limits repeated connection
attempts from the same source (ufw denies an IP that attempts to connect 6
or more times within a 30 second period). This is useful for
internet-facing services like SSH where you want to allow access but guard
against brute-force attempts.

`ufw_forward_policy` controls whether traffic _forwarded through_ the host
(as opposed to traffic destined for the host itself) is allowed. UFW's
default is to drop forwarded traffic, which breaks CNI plugins such as
Calico on Kubernetes nodes that route pod traffic through the host. Set
this to `"allow"` on such nodes -- see [Canonical Kubernetes' UFW
guide](https://documentation.ubuntu.com/canonical-kubernetes/main/snap/howto/networking/ufw/)
for the port allowlist Calico also needs (added via `ufw_allow_rules`).

`ufw_route_rules` is a less permissive alternative to `ufw_forward_policy`:
rather than allowing all forwarded traffic, it scopes forwarding to
specific subnet pairs, e.g. `{ src: "10.1.0.0/16", dest: "10.0.20.0/24" }`
for `ufw route allow from 10.1.0.0/16 to 10.0.20.0/24`. Leaving `src` or
`dest` unset (or `"any"`) scopes the rule to only the side that's set, but
at least one of them must be set to something other than `"any"` -- a rule
with both left as `"any"` is what `ufw_forward_policy` is for.

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
        ufw_route_rules:
          - {
              src: "10.1.0.0/16",
              dest: "10.0.20.0/24",
              comment: "Calico pod -> service subnet",
            }
```
