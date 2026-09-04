# ctsc.sys.docker

Installs Docker Engine (`docker-ce`, `docker-ce-cli`, `containerd.io`,
`docker-buildx-plugin`, `docker-compose-plugin`) from Docker's official apt
repository, following [Docker's install instructions](https://docs.docker.com/engine/install/ubuntu/).
Conflicting distro packages (`docker.io`, `docker-compose`, `podman-docker`,
etc.) are removed first, and the `docker` service is enabled/started.

Optionally (`docker_configure_ufw: true`) also inserts a `DOCKER-USER`
iptables chain into `/etc/ufw/after.rules` and enables `ufw`. Docker
manipulates iptables directly to publish container ports, which bypasses
ufw's rules entirely -- this chain restores ufw's control over inbound
traffic to published container ports.

**Caution:** if `docker_configure_ufw` is true, this role enables ufw. If
ufw isn't already active with an allowlist that includes SSH, enabling it
here can lock you out of the host. Run `ctsc.sys.ufw` first (with
`ufw_allow_rules` including SSH) whenever pairing it with this role.

## Variables

| Variable              | Default | Description                                                    |
| ---------------------- | ------- | --------------------------------------------------------------- |
| `docker_configure_ufw` | `false` | Add the `DOCKER-USER` ufw/iptables fix and enable ufw.         |

## Example

```yaml
- hosts: all
  become: true
  roles:
    - role: ctsc.sys.ufw
      vars:
        ufw_allow_rules:
          - { port: "ssh", proto: tcp, comment: "SSH" }
    - role: ctsc.sys.docker
      vars:
        docker_configure_ufw: true
```
