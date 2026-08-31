FROM ubuntu:24.04

ARG DEBIAN_FRONTEND=noninteractive

# Shared base image for every role scenario under extensions/molecule/. Installs
# the union of packages any scenario's converge/verify needs (curl + iproute2
# are only exercised by ufw's outbound-request check) so scenarios can point
# their molecule.yml at this single file via `dockerfile: ../Dockerfile`.
RUN apt-get update && \
    apt-get install --yes python3 systemd sudo curl iproute2 && \
    apt-get clean

CMD ["/lib/systemd/systemd"]
