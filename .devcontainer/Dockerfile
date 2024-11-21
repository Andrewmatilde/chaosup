FROM mcr.microsoft.com/devcontainers/base:bullseye
RUN apt-get update
# Install CNI Plugins
RUN wget -q https://github.com/opencontainers/runc/releases/download/v1.2.1/runc.amd64
RUN install -m 755 runc.amd64 /usr/local/sbin/runc
RUN wget -q  https://github.com/containernetworking/plugins/releases/download/v1.6.0/cni-plugins-linux-amd64-v1.6.0.tgz
RUN mkdir -p /opt/cni/bin
RUN tar -C /opt/cni/bin -xzf cni-plugins-linux-amd64-v1.6.0.tgz