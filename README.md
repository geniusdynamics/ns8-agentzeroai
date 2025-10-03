# ns8-agentzeroai

A NethServer 8 module for integrating AgentZero AI, providing AI-powered automation and assistance within the NethServer ecosystem.

## Description

This module deploys AgentZero AI as a service in NethServer 8, allowing users to leverage AI capabilities for various tasks. It includes a web interface and backend services configured with Traefik for routing and SSL support.

## Features

- AI-powered agent for automation and assistance
- Web-based user interface
- Database integration with MariaDB
- Automatic SSL certificate management with Let's Encrypt
- Smarthost configuration discovery
- Multi-language support via Weblate

## Requirements

- NethServer 8 core system
- Docker/Podman for containerization
- Access to a domain for hosting

## Installation

Instantiate the module with:

    add-module ghcr.io/geniusdynamics/agentzeroai:latest 1

The output of the command will return the instance name.
Output example:

    {"module_id": "agentzeroai1", "image_name": "agentzeroai", "image_url": "ghcr.io/geniusdynamics/agentzeroai:latest"}

## Configuration

Launch `configure-module`, by setting the following parameters:

- `host`: a fully qualified domain name for the application
- `http2https`: enable or disable HTTP to HTTPS redirection (true/false)
- `lets_encrypt`: enable or disable Let's Encrypt certificate (true/false)

Example:

```
api-cli run configure-module --agent module/agentzeroai1 --data - <<EOF
{
  "host": "agentzeroai.domain.com",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

The above command will:

- start and configure the agentzeroai instance
- configure a virtual host for Traefik to access the instance

## Get the Configuration

You can retrieve the configuration with:

```
api-cli run get-configuration --agent module/agentzeroai1
```

## Update

To update the module to a new version

```bash
api-cli run update-module --data '{
  "module_url": "ghcr.io/geniusdynamics/agentzeroai:latest",
  "instances": ["agentzeroai1"],
  "force": true
}'

```

## Uninstall

To uninstall the instance:

    remove-module --no-preserve agentzeroai1

## Smarthost Setting Discovery

Some configuration settings, like the smarthost setup, are not part of the `configure-module` action input: they are discovered by looking at some Redis keys. To ensure the module is always up-to-date with the centralized [smarthost setup](https://nethserver.github.io/ns8-core/core/smarthost/) every time agentzeroai starts, the command `bin/discover-smarthost` runs and refreshes the `state/smarthost.env` file with fresh values from Redis.

Furthermore, if smarthost setup is changed when agentzeroai is already running, the event handler `events/smarthost-changed/10reload_services` restarts the main module service.

See also the `systemd/user/agentzeroai.service` file.

This setting discovery is just an example to understand how the module is expected to work: it can be rewritten or discarded completely.

## Debug

Some CLI commands are needed to debug:

- The module runs under an agent that initiates a lot of environment variables (in `/home/agentzeroai1/.config/state`). It could be nice to verify them on the root terminal:

  ```
  runagent -m agentzeroai1 env
  ```

- You can become runagent for testing scripts and initiate all environment variables:

  ```
  runagent -m agentzeroai1
  ```

  The path becomes:

  ```
  echo $PATH
  /home/agentzeroai1/.config/bin:/usr/local/agent/pyenv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/
  ```

- If you want to debug a container or see environment inside:

  ```
  runagent -m agentzeroai1
  podman ps
  ```

  Example output:

  ```
  CONTAINER ID  IMAGE                                      COMMAND               CREATED        STATUS        PORTS                    NAMES
  d292c6ff28e9  localhost/podman-pause:4.6.1-1702418000                          9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  80b8de25945f-infra
  d8df02bf6f4a  docker.io/library/mariadb:10.11.5          --character-set-s...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  mariadb-app
  9e58e5bd676f  docker.io/library/nginx:stable-alpine3.17  nginx -g daemon o...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  agentzeroai-app
  ```

- You can see what environment variables are inside the container:

  ```
  podman exec agentzeroai-app env
  ```

  Example output:

  ```
  PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
  TERM=xterm
  PKG_RELEASE=1
  MARIADB_DB_HOST=127.0.0.1
  MARIADB_DB_NAME=agentzeroai
  MARIADB_IMAGE=docker.io/mariadb:10.11.5
  MARIADB_DB_TYPE=mysql
  container=podman
  NGINX_VERSION=1.24.0
  NJS_VERSION=0.7.12
  MARIADB_DB_USER=agentzeroai
  MARIADB_DB_PASSWORD=agentzeroai
  MARIADB_DB_PORT=3306
  HOME=/root
  ```

- You can run a shell inside the container:

  ```
  podman exec -ti agentzeroai-app sh
  ```

## Testing

Test the module using the `test-module.sh` script:

    ./test-module.sh <NODE_ADDR> ghcr.io/geniusdynamics/agentzeroai:latest

The tests are made using [Robot Framework](https://robotframework.org/).

## UI Translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To set up the translation process:

- Add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- Add your repository to [hosted.weblate.org](https://hosted.weblate.org) or ask a NethServer developer to add it to the ns8 Weblate project
