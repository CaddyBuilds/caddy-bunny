# Caddy Bunny

[![Build](https://github.com/caddybuilds/caddy-bunny/actions/workflows/check-caddy-release.yml/badge.svg)](https://github.com/caddybuilds/caddy-bunny/actions/workflows/check-caddy-release.yml)
[![License: MIT](https://img.shields.io/github/license/caddybuilds/caddy-bunny)](LICENSE)

> A fully integrated Caddy Docker image featuring Bunny DNS-01 ACME validation

Automated, multi-platform Docker builds of [Caddy](https://caddyserver.com/) with the [Bunny.net](https://bunny.net/) DNS module baked in.

## Table of Contents

- [Features](#features)
- [Quick Start](#quick-start)
- [Included Modules](#included-modules)
- [Usage](#usage)
  - [Docker Pull](#docker-pull)
  - [Caddyfile Examples](#caddyfile-examples)
    - [Global DNS Configuration](#global-dns-configuration)
    - [Per-Site DNS Configuration](#per-site-dns-configuration)
- [Configuration](#configuration)
  - [Getting a Bunny API Key](#getting-a-bunny-api-key)
  - [ACME DNS Challenge](#acme-dns-challenge)
- [Tags](#tags)
  - [Default or slim?](#default-or-slim)
  - [Why `alpine` tags point to the same image](#why-alpine-tags-point-to-the-same-image)
  - [Why did a digest change?](#why-did-a-digest-change)
- [Platform Support](#platform-support)
  - [Alpine](#alpine)
  - [Raspberry Pi](#raspberry-pi)
- [Troubleshooting](#troubleshooting)
  - [Debug Logging](#debug-logging)
  - [Verifying Modules](#verifying-modules)
  - [Official Resources](#official-resources)
- [How It Works](#how-it-works)
- [Building Your Own Image](#building-your-own-image)
  - [Prerequisites](#prerequisites)
  - [Setup Instructions](#setup-instructions)
- [Contributing](#contributing)
- [License](#license)

## Features

- **Automated Builds** -- Daily checks for new Caddy releases with automatic image builds.
- **Module Update Detection** -- Tracks upstream module commits and rebuilds when they change.
- **Base Image Tracking** -- Rebuilds when the base image is republished with OS or Go toolchain fixes.
- **Bunny DNS-01 Integration** -- Automatic SSL certificate management via DNS challenges.
- **Multi-Platform** -- Supports `amd64`, `arm64`, `arm/v7` (Raspberry Pi), `ppc64le`, and `s390x`.
- **Alpine-Based** -- Built on the official Caddy image, which uses Alpine Linux. A `slim` variant drops the stock binary.
- **Fork-Friendly** -- Image names derived automatically from your GitHub repository.

## Quick Start

Pull the image and set up a `docker-compose.yml`:

```yaml
services:
  caddy:
    image: ghcr.io/caddybuilds/caddy-bunny:latest
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - $PWD/site:/srv
      - caddy_data:/data
      - caddy_config:/config
    environment:
      - BUNNY_API_KEY=your_bunny_api_key

volumes:
  caddy_data:
    external: true
  caddy_config:
```

> Defining the data volume as [external](https://docs.docker.com/compose/compose-file/compose-file-v3/#external) ensures `docker-compose down` does not delete it. Create it manually with `docker volume create caddy_data`.

Replace `your_bunny_api_key` with your Bunny.net API key. See [Getting a Bunny API Key](#getting-a-bunny-api-key) for where to find it, and [Caddyfile Examples](#caddyfile-examples) for configuration samples.

## Included Modules

This module is compiled into Caddy at build time using [xcaddy](https://github.com/caddyserver/xcaddy). No additional installation is needed.

| Module | Purpose | Repository |
|--------|---------|------------|
| `caddy-dns/bunny` | DNS-01 ACME challenge provider for Bunny.net | [caddy-dns/bunny](https://github.com/caddy-dns/bunny) |

## Usage

### Docker Pull

```sh
docker pull ghcr.io/caddybuilds/caddy-bunny:latest
docker pull caddybuilds/caddy-bunny:latest
# slim
docker pull ghcr.io/caddybuilds/caddy-bunny:slim
docker pull caddybuilds/caddy-bunny:slim
```

See [Quick Start](#quick-start) for a full docker-compose setup and [Tags](#tags) for every tag.

### Caddyfile Examples

#### Global DNS Configuration

Set the ACME DNS challenge provider globally so it applies to all sites:

```
{
  acme_dns bunny {env.BUNNY_API_KEY}
}

example.com {
    root * /usr/share/caddy
    file_server
    encode gzip
}

another-example.com {
    reverse_proxy localhost:8080
}
```

#### Per-Site DNS Configuration

Configure the DNS challenge individually per site:

```
example.com {
    root * /usr/share/caddy
    file_server
    encode gzip

    tls {
        dns bunny {env.BUNNY_API_KEY}
    }
}
```

## Configuration

### Getting a Bunny API Key

Your Bunny.net account already has an API key. Retrieve it and pass it to Caddy:

1. **Find your API key** in the Bunny.net dashboard under [Account settings](https://dash.bunny.net/account/settings). Bunny's [API key docs](https://docs.bunny.net/account/api-keys) explain the details.

2. **Set the environment variable**:
   - Add `BUNNY_API_KEY` to your deployment environment. See [Quick Start](#quick-start) for the docker-compose example.

You can also put the key directly in the Caddyfile instead of `{env.BUNNY_API_KEY}`, but an environment variable keeps it out of your config files.

### ACME DNS Challenge

To configure the [ACME DNS challenge](https://caddyserver.com/docs/automatic-https#dns-challenge) provider for all ACME transactions, add the following to your Caddyfile:

```
{
   acme_dns bunny {env.BUNNY_API_KEY}
}
```

This is equivalent to specifying the provider in the [tls directive's ACME issuer](https://caddyserver.com/docs/caddyfile/directives/tls#acme) configuration. See [Caddyfile Examples](#caddyfile-examples) for full samples.

## Tags

The image is available on [GitHub Container Registry](https://github.com/caddybuilds/caddy-bunny/pkgs/container/caddy-bunny) and Docker Hub. There are two images, default and slim, each under a few tags:

| Tags | Image |
|------|-------|
| `latest`, `<version>`, `<major>.<minor>`, `<major>` | Default |
| `alpine`, `<version>-alpine`, `<major>.<minor>-alpine`, `<major>-alpine` | Default (same image, see below) |
| `slim`, `<version>-slim`, `<major>.<minor>-slim`, `<major>-slim` | Slim |

`<version>` pins a release (e.g. `docker pull ghcr.io/caddybuilds/caddy-bunny:2.11.7`), `<major>.<minor>` follows the latest patch (e.g. `2.11`) and `<major>` the latest release (e.g. `2`). Slim tags exist from `2.11.7` onward.

### Default or slim?

Both run the same Caddy binary with the same modules. They differ only in the image underneath.

| | Default | Slim |
|---|---|---|
| Built on | The official [Caddy image](https://hub.docker.com/_/caddy) | [caddy-base](https://github.com/CaddyBuilds/caddy-base#the-caddy-base-image): the official image without its Caddy binary |
| Download size (amd64) | about 43 MB | about 24 MB |
| Caddy binaries inside | Two: the stock one stays in a lower layer, ours replaces it on top | One: ours |
| Rebuilt for base fixes when | The official Caddy image is republished | Alpine is republished (caddy-base rebuilds, then slim follows) |

Everything else is the same: Alpine, packages, default Caddyfile, `/data` and `/config`, ports, command, platforms, and permission to bind ports 80 and 443 as a non-root user.

Use **slim** for a smaller image. Use **default** if you want to stay on the Docker Official Image exactly as Caddy ships it.

### Why `alpine` tags point to the same image

This follows the official Caddy image. On Linux, `caddy:latest`, `caddy:2.11.7` and `caddy:2.11.7-alpine` are all the same Alpine image. The plain tags also include Windows builds, which is the only reason both exist. We only build Linux images, so the `-alpine` tags are aliases: a setup that uses `caddy:2.11.7-alpine` can switch to `caddy-bunny:2.11.7-alpine` without changing anything else.

### Why did a digest change?

Images are rebuilt without a new Caddy version when the module gets new commits or a base image is republished with OS or Go fixes (the official Caddy image for the default tags, caddy-base for `slim`). Each rebuild is logged under **Rebuilds** in that version's [GitHub Release](https://github.com/caddybuilds/caddy-bunny/releases), and the reason is stored in the image labels:

```sh
docker inspect --format '{{ index .Config.Labels "caddy-build.reason" }}: {{ index .Config.Labels "caddy-build.details" }}' ghcr.io/caddybuilds/caddy-bunny:latest
```

## Platform Support

The image is built for the following platforms:

- **linux/amd64** -- Standard x86_64 (desktops, servers)
- **linux/arm64** -- ARM 64-bit (modern ARM servers, Apple Silicon via emulation)
- **linux/arm/v7** -- ARM 32-bit (Raspberry Pi 3+)
- **linux/ppc64le** -- 64-bit PowerPC Little Endian
- **linux/s390x** -- 64-bit IBM System z

### Alpine

Both images are built on [Alpine Linux](https://alpinelinux.org/): the default through the official Caddy image, slim through caddy-base. Alpine uses musl libc instead of glibc, which may cause compatibility issues with some software. Additional tools (git, bash, etc.) are not included by default.

### Raspberry Pi

The `linux/arm/v7` build supports Raspberry Pi 3 and later. No special configuration is needed beyond having Docker installed on a compatible OS (such as Raspberry Pi OS).

## Troubleshooting

### Debug Logging

The Bunny module logs additional information when Caddy's global debug mode is on. Enable it while you investigate a failing challenge:

```
{
  debug
  acme_dns bunny {env.BUNNY_API_KEY}
}
```

### Verifying Modules

To confirm the Bunny module is loaded:

```sh
docker run --rm ghcr.io/caddybuilds/caddy-bunny:latest caddy list-modules | grep dns.providers.bunny
```

### Official Resources

- [caddy-dns/bunny](https://github.com/caddy-dns/bunny)
- [Bunny.net API keys](https://docs.bunny.net/account/api-keys)
- [Caddy ACME DNS challenge docs](https://caddyserver.com/docs/automatic-https#dns-challenge)

## How It Works

The build pipeline is fully automated via GitHub Actions and shared with the other CaddyBuilds images through [caddy-base](https://github.com/CaddyBuilds/caddy-base). This repo holds only what is specific to this image: the `Dockerfile` (module list), `caddy-image.json` (title, description, verified module) and two thin workflows that call the shared ones.

1. **Daily release check** -- A scheduled workflow queries the GitHub API for new [Caddy releases](https://github.com/caddyserver/caddy/releases) and verifies the official Docker image is available with all required platforms before triggering a build.

2. **Module update detection** -- The pipeline reads the module list from the `Dockerfile` and tracks each module's latest commit SHA in `module-versions.json`. If any module has new commits, a rebuild is triggered even without a new Caddy release.

3. **Base image tracking** -- Docker Official Images republish existing Caddy versions for base OS and Go toolchain fixes, and caddy-base is rebuilt when Alpine is republished. The check compares each base image with when ours was built, and rebuilds if the base is newer.

4. **Multi-platform build** -- The shared workflow builds the default and slim images for all supported architectures. Images are pushed to GitHub Container Registry (primary) and optionally Docker Hub.

5. **Fork-friendly** -- Image names are derived from the `GITHUB_REPOSITORY` environment variable, so forks produce correctly named images without any configuration changes.

6. **Failure notifications** -- If a build fails, a GitHub issue is automatically created for visibility.

## Building Your Own Image

### Prerequisites

- GitHub account
- Docker Hub account (optional, for pushing to Docker Hub)
- GitHub Secrets configured:
  - `GITHUB_TOKEN` (automatically available in GitHub Actions)
  - `DOCKERHUB_USERNAME` (optional)
  - `DOCKERHUB_TOKEN` (optional)
  - `DOCKERHUB_REPOSITORY_NAME` (optional, to customize the Docker Hub repository name)

### Setup Instructions

1. **[Fork this repository](https://github.com/caddybuilds/caddy-bunny/fork)** to your GitHub account.

2. **Set up GitHub Secrets** (optional, for Docker Hub):
   - Navigate to `Settings` > `Secrets and variables` > `Actions`.
   - Add `DOCKERHUB_USERNAME`, `DOCKERHUB_TOKEN`, and optionally `DOCKERHUB_REPOSITORY_NAME`.

3. **Enable GitHub Actions**:
   - Go to the `Actions` tab. If prompted, click "I understand my workflows, go ahead and enable them".

4. **Customize the modules** (optional):
   - Add or change `--with` lines in the `Dockerfile`; the daily check follows the list from there. Adjust the title and description in `caddy-image.json` if you like.

5. **Trigger a build** (optional):
   - In the `Actions` tab, select `Build and Push Docker Image` and click `Run workflow`. The optional note is recorded in the release notes.

6. **Pull your image**:
   ```sh
   docker pull ghcr.io/YOUR_GITHUB_USERNAME/YOUR_REPOSITORY_NAME:latest
   ```

Use the image in your docker-compose setup as shown in the [Quick Start](#quick-start), replacing the image name with your own.

## Contributing

Feel free to open issues or submit pull requests if you have any improvements or bug fixes.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
