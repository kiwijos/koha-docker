FROM debian:bookworm-20250428-slim

# Avoid prompts from apt
ENV DEBIAN_FRONTEND=noninteractive

# Following a version number is recommended 
ARG KOHA_VERSION=24.11

# S6 Overlay is a process supervisor that
# allows us to manage multiple processes in the container
ARG S6_OVERLAY_VERSION=3.2.0.2
ARG S6_OVERLAY_ARCH=x86_64

# Initial setup (some packages are needed to get the keys etc.)
RUN apt-get update && apt-get install -y \
    wget \
    gnupg2 \
    apt-transport-https \
    xz-utils \
    && rm -rf /var/cache/apt/archives/* \
    && rm -rf /var/lib/apt/lists/*

# Install S6 Overlay
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-noarch.tar.xz /tmp
ADD https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-${S6_OVERLAY_ARCH}.tar.xz /tmp

RUN tar -C / -Jxpf /tmp/s6-overlay-noarch.tar.xz \
    && tar -C / -Jxpf /tmp/s6-overlay-${S6_OVERLAY_ARCH}.tar.xz \
    && rm /tmp/s6-overlay-noarch.tar.xz \
    && rm /tmp/s6-overlay-${S6_OVERLAY_ARCH}.tar.xz

# Get keys to allow the package manager to securely
# download and verify packages from the Koha repository
RUN mkdir -p /etc/apt/keyrings \
    && wget -q -O- https://debian.koha-community.org/koha/gpg.asc | gpg --dearmor -o /etc/apt/keyrings/koha.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/koha.gpg] https://debian.koha-community.org/koha ${KOHA_VERSION} main" | tee /etc/apt/sources.list.d/koha.list

# Install Koha and its dependencies
RUN apt-get update && apt-get install -y \
    koha-core \
    idzebra-2.0 \
    apache2 \
    libapache2-mpm-itk \
    && /etc/init.d/koha-core stop \
    && rm -rf /var/cache/apt/archives/* \
    && rm -rf /var/lib/apt/lists/*

# Configure Apache
#
# - Enable the necessary modules
# - Disable the default site
# - Listen on port 8080 and 8081
# - Set the server name to suppress warnings
RUN a2enmod rewrite cgi headers proxy_http \
    && a2dissite 000-default \ 
    && echo "Listen 8081\nListen 8080" > /etc/apache2/ports.conf \
    && echo "ServerName localhost" | tee /etc/apache2/conf-available/fqdn.conf \ 
    && a2enconf fqdn

# Add our custom configuration files and scripts
COPY --chown=0:0 --chmod=755 rootfs/ /
COPY --chown=0:0 --chmod=755 docker /docker

WORKDIR /docker

EXPOSE 8080 8081

# Run S6 Overlay init process
ENTRYPOINT ["/init"]
