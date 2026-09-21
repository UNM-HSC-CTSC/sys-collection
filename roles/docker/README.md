# ctsc.sys.docker

Installs Docker Engine (`docker-ce`, `docker-ce-cli`, `containerd.io`,
`docker-buildx-plugin`, `docker-compose-plugin`) from Docker's official apt
repository, following [Docker's install instructions](https://docs.docker.com/engine/install/ubuntu/).
Conflicting distro packages (`docker.io`, `docker-compose`, `podman-docker`,
etc.) are removed first, and the `docker` service is enabled/started.

Optionally (`docker_configure_ufw: true`) also inserts a `DOCKER-USER`
iptables chain into `/etc/ufw/after.rules` and enables `ufw`. Docker
manipulates iptables directly to publish container ports, which bypasses
ufw's `allow` rules entirely -- this chain restores ufw's control over
inbound traffic to published container ports.

**Inbound traffic to Docker bridge networks is default-deny.** A new
connection into a Docker bridge (i.e. to any published container port) is
dropped, from any source, unless a `ufw route allow` rule accepts it first.
Traffic routed through the host that is not entering a Docker bridge
(e.g. Kubernetes/Calico pod traffic) is not filtered by this chain, and
containers' own outbound traffic is left to Docker. Notes:

- Use routed rules, not plain `ufw allow`: published ports are DNAT'd and
  forwarded, so `ufw allow` (INPUT chain) rules never apply to them. With
  `ctsc.sys.ufw` that means `ufw_route_rules`, e.g.
  `{ src: "192.168.100.2", port: "8000", proto: tcp }`.
- Match the **container** port, not the published host port
  (`-p 8080:80` is opened with `port: "80"`).
- This differs from upstream [ufw-docker](https://github.com/chaifeng/ufw-docker),
  which exempts every RFC 1918 source range (`10/8`, `172.16/12`,
  `192.168/16`) from filtering, so any host on a private network can reach
  any published port. That exemption was removed here: on hosts that are
  already behind a network firewall it provided no protection while
  making ufw rules ineffective for private sources.
- Existing published ports stop being reachable (except from sources you
  allow) once this is applied. List them first with
  `docker ps --format '{{.Names}} {{.Ports}}'` and add matching route rules.

**Caution:** if `docker_configure_ufw` is true, this role enables ufw. If
ufw isn't already active with an allowlist that includes SSH, enabling it
here can lock you out of the host. Run `ctsc.sys.ufw` first (with
`ufw_allow_rules` including SSH) whenever pairing it with this role.

## Variables

| Variable                       | Default             | Description                                                                                                                              |
| ------------------------------ | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `docker_configure_ufw`         | `false`             | Add the `DOCKER-USER` ufw/iptables fix and enable ufw.                                                                                   |
| `docker_ufw_bridge_interfaces` | `[docker0, "br-+"]` | Bridge interfaces whose inbound traffic is default-denied. iptables matches: a trailing `+` is a wildcard. Add custom bridge names here. |
| `docker_ufw_block_container_egress` | `false` | Also drop new outbound connections from containers on those bridges to anything beyond their own bridge. Open exceptions with `ufw route allow from <container subnet> to <destination> port N`. Does not filter container-to-host traffic. |

## Example

```yaml
- hosts: all
  become: true
  roles:
    - role: ctsc.sys.ufw
      vars:
        ufw_allow_rules:
          - { port: "ssh", proto: tcp, comment: "SSH" }
        ufw_route_rules:
          # container port 80 (published however you like), from one host only
          - { src: "192.168.100.2", port: "80", proto: tcp, comment: "web container from peer" }
    - role: ctsc.sys.docker
      vars:
        docker_configure_ufw: true
```
