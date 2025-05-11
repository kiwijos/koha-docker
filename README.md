# Koha Docker

A Docker setup for running Koha, the open-source integrated library system.

## Overview

This project provides a containerized environment for running Koha using Docker Compose. It includes these services:

- **Koha**: The main application container running Koha ILS on Debian with Apache
- **MariaDB**: Database server for storing Koha data
- **Memcached**: Caching service to improve Koha performance
- **RabbitMQ**: Message broker for background tasks

The setup uses S6 Overlay for process supervision within the Koha container, allowing fine-grained control over multiple processes.

## Quick Start

1. Clone this repository:
   ```
   git clone <repository-url>
   cd koha-docker
   ```

2. Start the containers:
   ```
   docker-compose up -d
   ```

(The `-d` flag runs the containers in detached mode. It might be educational to start without `-d` and watch the logs.)

3. Access Koha Staff interface at:
    ```
    http://localhost:8080
    ```

4. Login with the default credentials:
   - Username: `koha_<instance>` (e.g., `koha_test`)
   - Password: `supersecret`

5. Follow the on-screen instructions to set up your Koha instance.

6. Now you can access the Koha OPAC (Online Public Access Catalog) at:
   ```
   http://localhost:8081
   ```

## Configuration

All configuration options are available in the [`.env` file](.env). You can modify this file directly or create a custom `.env` file to override specific settings.

Some users may also want to set environment variables temporarily with the `--env` flag for quick testing.

### Key Configuration Options

#### Koha Instance
```
KOHA_INSTANCE=test                # Name of the Koha instance and database
```

Tip: Make the name distinctive, perhaps by using the name of your library, institution, or a project.

#### Network Configuration
```
KOHA_INTRANET_PORT=8080           # Port for staff interface
KOHA_OPAC_PORT=8081               # Port for public interface
```

#### Database Settings
```
MYSQL_PASSWORD=supersecret        # Change this for production!
MYSQL_ROOT_PASSWORD=root          # Change this for production!
```

#### Localization
```
KOHA_LANGS=sv-SE ar-Arab          # Languages to install besides English
TZ=Europe/Stockholm               # Timezone
```

#### Service Versions
```
MEMCACHED_VERSION=1.6.38-bookworm
RABBITMQ_VERSION=4.1.0-alpine
MARIADB_VERSION=10.6.21-focal
```

## Service Information

### Koha Container

The main Koha container runs the ILS software with the following components:

- **Apache Web Server**: Serves both staff and public interfaces
- **Zebra**: Handles search and indexing for the catalog
- **Plack**: Improves web server performance
- **Cron**: Manages scheduled tasks
- **Workers**: Process background jobs

The container uses S6 Overlay to manage these multiple processes efficiently.

### Database (MariaDB)

Stores all Koha data:

- Bibliographic records
- Patron information
- System configuration
- And more...

Data is persisted in a Docker volume.

To start fresh, you can remove the volume:
```
docker-compose down --volumes
```

MariaDB automatically creates a database and user for Koha on image startup. The names are derived from the `KOHA_INSTANCE` variable and should not be changed directly.

### Memcached

Can improve Koha performance by caching.

Currently, the default settings are used. For a full list of options, refer to the official documentation,
or run `memcached -h` or `man memcached` inside the container.

```
# In docker-compose.yml
memcached:
  command:
    - --conn-limit=1024
    - --memory-limit=64
    - --threads=4   
```

### RabbitMQ

Streams messages to manage background tasks. It talks with Koha using the STOMP protocol.

## Additional Configuration

### Custom Languages

To add additional languages:

1. Edit the `KOHA_LANGS` variable in `default.env`
2. Restart the containers

## Maintenance

### Updating Koha

To update to a newer version:

Edit the `KOHA_VERSION` in `koha/Dockerfile` and rebuild the containers:
```
docker-compose build --no-cache
docker-compose up -d
```
You can also specify a different version temporarily using the `--build-arg` flag:
```
docker-compose build --build-arg KOHA_VERSION=<new-version>
```

## Development

The project structure is currently organized as follows:

```
koha-docker/
├── default.env                # Environment variables
├── docker-compose.yml         # Service definitions
├── koha/
│   ├── Dockerfile             # Koha container definition
│   ├── docker/                # Configuration files etc.
│   └── rootfs/                # Overlay filesystem
└── rabbitmq-plugins    
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request or send me an email with suggestions.

## License

This project is licensed under GPL3 - see the [LICENSE file](LICENSE) for details.

Using GPL3 ensures we're complying with the licenses of Koha and the previous Docker implementations we're building on. It maintains the same open-source spirit.

## Acknowledgments

- Koha Community for the continuous development of Koha
- This implementation is based on two other Koha + Docker implementations:
  - [teogramm/koha-docker](https://github.com/teogramm/koha-docker)
  - [koha-community/docker/koha-docker](https://gitlab.com/koha-community/docker/koha-docker/-/tree/main?ref_type=heads)
- Docker and container ecosystem maintainers
