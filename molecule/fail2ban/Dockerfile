FROM ubuntu:24.04

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install --yes python3 systemd && \
    apt-get clean

CMD ["/lib/systemd/systemd"]
