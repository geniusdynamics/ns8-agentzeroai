# ns8-agentzeroai

A NethServer 8 module for integrating AgentZero AI, providing AI-powered automation and assistance within the NethServer ecosystem.

## Description

This module deploys AgentZero AI as a service in NethServer 8, allowing users to leverage AI capabilities for various tasks. It includes a web interface and backend services configured with Traefik for routing and SSL support.

## Features

- Agent Zero autonomous AI agent with its web interface
- Virtual host routing through Traefik, with optional Let's Encrypt certificate
- Backup, restore, clone and move of the instance with all its user data
- Multi-language support via Weblate

## Requirements

- NethServer 8
- A host name (FQDN) that resolves to the node
- An API key for at least one LLM provider, set from the Agent Zero web interface

## Security

Agent Zero runs arbitrary code inside its container and starts **without a
login**. Right after the first configuration, open the web interface and set a
user name and password in the *Authentication* section of the Agent Zero settings.
Until then, anyone who can reach the virtual host controls the agent.

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

## Data, backup and update

Only `/a0/usr` is persistent. It lives in the `agentzero-usr` volume and holds
the Agent Zero settings, `usr/.env` (API keys, login), chats, memory, projects,
skills, plugins and the work directory. The rest of `/a0` comes from the
container image, so `update-module` delivers the new Agent Zero release.
Update Agent Zero through NethServer, not with the in-app self update: changes
made by the self update are lost when the container restarts.

The backup includes the `agentzero-usr` volume. Restore, clone and move bring
it back and configure the Traefik route again.

Module releases up to 2.0.x kept the whole `/a0` directory in the
`agent_data_a` volume, which froze the Agent Zero code at the first installed
release. On update, and when restoring a backup taken by such a release,
`bin/migrate-usr-volume` copies `/a0/usr` of the old volume to
`agentzero-usr`. The old `agent_data_a` and `agentdata` volumes are left in
place: once the updated instance works, remove them with

    runagent -m agentzeroai1 podman volume rm agent_data_a agentdata

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

  The pod runs the `agentzero-app` container, from the
  `docker.io/agent0ai/agent-zero` image.

- You can see what environment variables are inside the container:

  ```
  podman exec agentzero-app env
  ```

- You can run a shell inside the container:

  ```
  podman exec -ti agentzero-app bash
  ```

## Testing

The [Robot Framework](https://robotframework.org/) suite in `tests/` runs on
GitHub Actions after each image build, on QEMU virtual machines, through the
shared NethServer
[test-module-qemu](https://github.com/NethServer/ns8-github-actions/blob/v1/docs/test-module-qemu.md)
workflow. The `install` scenario tests a clean installation; the `update`
scenario installs `ghcr.io/geniusdynamics/agentzeroai:latest`, updates it to
the image under test and checks the user data survived.

## UI Translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To set up the translation process:

- Add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- Add your repository to [hosted.weblate.org](https://hosted.weblate.org) or ask a NethServer developer to add it to the ns8 Weblate project
