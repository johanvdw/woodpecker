# Setup

A Woodpecker deployment consists of two parts:
- A server which is the heart of Woodpecker and ships the webinterface.
- Next to one server you can deploy any number of agents which will run the pipelines.

> Each agent is able to process one pipeline step by default.
>
> If you have 4 agents installed and connected to the Woodpecker server, your system will process 4 builds in parallel.
>
> You can add more agents to increase the number of parallel builds or set the agent's `WOODPECKER_MAX_PROCS=1` environment variable to increase the number of parallel builds for that agent.

## Installation

You can install Woodpecker on multiple ways:
- Using [docker-compose](/docs/administration/setup#docker-compose) with the official [docker images](/docs/downloads#docker-images)
- By deploying to a [Kubernetes](/docs/administration/kubernetes) with manifests or Woodpeckers official Helm charts
- Using [binaries](/docs/downloads)

### docker-compose

The below [docker-compose](https://docs.docker.com/compose/) configuration can be used to start a Woodpecker server with a single agent.

It relies on a number of environment variables that you must set before running `docker-compose up`. The variables are described below.

```yaml
# docker-compose.yml
version: '3'

services:
  woodpecker-server:
    image: woodpeckerci/woodpecker-server:latest
    ports:
      - 8000:8000
    volumes:
      - woodpecker-server-data:/var/lib/woodpecker/
    environment:
      - WOODPECKER_OPEN=true
      - WOODPECKER_HOST=${WOODPECKER_HOST}
      - WOODPECKER_GITHUB=true
      - WOODPECKER_GITHUB_CLIENT=${WOODPECKER_GITHUB_CLIENT}
      - WOODPECKER_GITHUB_SECRET=${WOODPECKER_GITHUB_SECRET}
      - WOODPECKER_AGENT_SECRET=${WOODPECKER_AGENT_SECRET}

  woodpecker-agent:
    image: woodpeckerci/woodpecker-agent:latest
    command: agent
    restart: always
    depends_on:
      - woodpecker-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WOODPECKER_SERVER=woodpecker-server:9000
      - WOODPECKER_AGENT_SECRET=${WOODPECKER_AGENT_SECRET}

volumes:
  woodpecker-server-data:
```

Woodpecker needs to know its own address. You must therefore provide the public address of it in `<scheme>://<hostname>` format. Please omit trailing slashes:

```diff
# docker-compose.yml
version: '3'

services:
  woodpecker-server:
    [...]
    environment:
      - [...]
+     - WOODPECKER_HOST=${WOODPECKER_HOST}
```

As agents run pipeline steps as docker containers they require access to the host machine's Docker daemon:

```diff
# docker-compose.yml
version: '3'

services:
  [...]
  woodpecker-agent:
    [...]
+   volumes:
+     - /var/run/docker.sock:/var/run/docker.sock
```

Agents require the server address for agent-to-server communication:

```diff
# docker-compose.yml
version: '3'

services:
  woodpecker-agent:
    [...]
    environment:
+     - WOODPECKER_SERVER=woodpecker-server:9000
```

The server and agents use a shared secret to authenticate communication. This should be a random string of your choosing and should be kept private. You can generate such string with `openssl rand -hex 32`:

```diff
# docker-compose.yml
version: '3'

services:
  woodpecker-server:
    [...]
    environment:
      - [...]
+     - WOODPECKER_AGENT_SECRET=${WOODPECKER_AGENT_SECRET}
  woodpecker-agent:
    [...]
    environment:
      - [...]
+     - WOODPECKER_AGENT_SECRET=${WOODPECKER_AGENT_SECRET}
```

## Authentication

Authentication is done using OAuth and is delegated to one of multiple version control providers, configured using environment variables. The example above demonstrates basic GitHub integration.

See the complete reference for all supported version control systems [here](/docs/administration/vcs/overview).

## Database

By default Woodpecker uses a sqlite database which requires zero installation or configuration. See the [database settings](/docs/administration/database) page to further configure it or use MySQL or Postgres.

## SSL

Woodpecker supports ssl configuration by using Let's encrypt or by using own certificates. See the [SSL guide](/docs/administration/ssl).

## Metrics

A [Prometheus endpoint](/docs/administration/prometheus) is exposed.

## Behind a proxy

See the [proxy guide](/docs/administration/proxy) if you want to see a setup behind Apache, Nginx, Caddy or ngrok.
