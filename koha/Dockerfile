FROM debian:bookworm-20250428-slim

# Avoid prompts from apt
ENV DEBIAN_FRONTEND=noninteractive

# Following a version number is recommended 
ARG KOHA_VERSION=24.11

# Initial setup (some packages are needed to get the keys etc.)
RUN apt-get update && apt-get install -y \
    wget \
    gnupg2 \
    apt-transport-https \
    && rm -rf /var/cache/apt/archives/* \
    && rm -rf /var/lib/apt/lists/*

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
    && rm -rf /usr/share/koha/misc/translator/po/* \
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
COPY --chmod=755 docker /docker

WORKDIR /docker

EXPOSE 8080 8081

ENTRYPOINT [ "./scripts/prepare-koha" ]
