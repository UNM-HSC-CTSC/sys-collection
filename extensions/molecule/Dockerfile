# Ubuntu version this image is built from. Every scenario's molecule.yml
# builds this same Dockerfile twice -- once per supported Ubuntu release --
# passing UBUNTU_VERSION as a docker build arg, so every role gets verified
# against both.
ARG UBUNTU_VERSION=24.04
FROM ubuntu:${UBUNTU_VERSION}

ARG DEBIAN_FRONTEND=noninteractive

# Shared base image for every role scenario under extensions/molecule/. Installs
# the union of packages any scenario's converge/verify needs (curl + iproute2
# are only exercised by ufw's outbound-request check, rsyslog by fail2ban's
# sshd jail -- see below) so scenarios can point their molecule.yml at this
# single file via `dockerfile: ../Dockerfile`.
RUN apt-get update && \
    apt-get install --yes python3 systemd sudo curl iproute2 rsyslog && \
    apt-get clean

# A real Ubuntu Server host already has /var/log/auth.log by the time this
# collection's roles run on it: rsyslog plus routine PAM/cron activity from
# first boot keep it populated. This minimal test image never generates that
# boot-time chatter, so rsyslog would never lazily create the file, and
# fail2ban's sshd jail (which watches auth.log) would fail to start on Ubuntu
# 22.04's older fail2ban, which needs the file to already exist (24.04's
# fail2ban defaults to the systemd journal backend instead, so it doesn't hit
# this). Pre-touch it so both behave like a real host rather than this
# container's own boot-time idiosyncrasy.
RUN touch /var/log/auth.log && \
    chown syslog:adm /var/log/auth.log && \
    chmod 0640 /var/log/auth.log

CMD ["/lib/systemd/systemd"]
