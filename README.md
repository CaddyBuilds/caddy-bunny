
# Caddy Bunny
> A fully integrated Caddy Docker image featuring Bunny DNS-01 ACME validation

Deploy a hassle-free Caddy server with built-in support for Bunny DNS-01 ACME challenges. Streamline your SSL certificate management and ensure your server stays secure without manual updates, making it an effortless and reliable solution.


## Table of Contents

- [Features](#features)
- [Usage](#usage)
  - [Using the Pre-built Docker Image](#using-the-pre-built-docker-image)
- [Sample Caddyfile](#sample-caddyfile)
  - [Global Configuration (Use DNS Challenge for All Sites)](#global-configuration-use-dns-challenge-for-all-sites)
  - [Per-site Configuration](#per-site-configuration)
- [Configuration](#configuration)
  - [Managing your Bunny API key](#managing-your-bunny-api-key)
  - [ACME DNS Challenge Configuration](#acme-dns-challenge-configuration)
    - [Troubleshooting](#troubleshooting)
- [Tags](#tags)
  - [Default or slim?](#default-or-slim)
  - [Why `alpine` tags point to the same image](#why-alpine-tags-point-to-the-same-image)
- [Platform Support](#platform-support)
  - [Alpine](#alpine)
  - [Raspberry Pi Support](#raspberry-pi-support)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Usage](#usage-1)
- [Contributing](#contributing)
- [License](#license)

## Features

- **Automated Builds**: Automatically checks for new Caddy releases and builds Docker images.
- **Continuous Integration**: Utilizes GitHub Actions for seamless CI/CD.
- **Bunny DNS Integration**: Integrates Bunny DNS for automatic SSL certificate management.
- **Multi-Platform Support**: Builds images for multiple architectures, including `amd64`, `arm64`, `arm/v7` (Raspberry Pi), `ppc64le`, and `s390x` , ensuring compatibility across a wide range of devices and systems.
- **Module Update Detection**: Rebuilds when the Bunny DNS module gets new commits, not only on Caddy releases.
- **Base Image Tracking**: Rebuilds when the official Caddy image is republished for base OS or Go fixes.
- **Slim Variant**: `slim` tags carry one Caddy binary on an Alpine base, about 45% smaller to download.
- **Fork-Friendly**: Image names are derived from your GitHub repository; edit one line in the Dockerfile to change the modules.
- **Manual Trigger**: Allows manual triggering of the build process.
- **Open Source**: Contributions are welcome under the MIT License.

## Usage

### Using the Pre-built Docker Image

To use the pre-built Docker image, pull it from the GitHub Container Registry:

```sh
docker pull ghcr.io/caddybuilds/caddy-bunny:latest
docker pull caddybuilds/caddy-bunny:latest
# alpine
docker pull ghcr.io/caddybuilds/caddy-bunny:alpine
docker pull caddybuilds/caddy-bunny:alpine
```
You can use the image in your Docker setup. Here is an example `docker-compose.yml` file:
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
Defining the data volume as [external](https://docs.docker.com/compose/compose-file/compose-file-v3/#external) makes sure `docker-compose down` does not delete the volume. You may need to create it manually using `docker volume create caddy_data`.

Replace `your_bunny_api_key` with your actual Bunny API key.

## Sample Caddyfile
Here is a sample Caddyfile configuration to get you started. This configuration sets up the ACME DNS challenge provider to use Bunny and serves a simple static site.

### [Global Configuration (Use DNS Challenge for All Sites)](./Caddyfile_global)
In this configuration, the ACME DNS challenge provider is set globally, so it applies to all sites served by Caddy.

### [Per-site Configuration](./Caddyfile_per_site)

## Configuration
### Managing your Bunny API key

Your Bunny account already has an API key generated. Follow these steps to retrieve it and use it in your Caddy setup.

1. **Follow Bunny's docs to retrieve your API key**:
   - [https://docs.bunny.net/account/api-keys](https://docs.bunny.net/account/api-keys)

2. **Set the Environment Variable**:
   - In your deployment environment, set the environment variable `BUNNY_API_KEY` to the value of the API key

For example, in a Docker environment, you can set this environment variable in your `docker-compose.yml` file:

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
Defining the data volume as [external](https://docs.docker.com/compose/compose-file/compose-file-v3/#external) makes sure `docker-compose down` does not delete the volume. You may need to create it manually using `docker volume create caddy_data`.

Replace `your_bunny_api_key` with your actual API key.

### ACME DNS Challenge Configuration
To configure the [ACME DNS challenge](https://caddyserver.com/docs/automatic-https#dns-challenge) provider for all ACME transactions, add the following to your Caddyfile:
```
{
  acme_dns bunny {env.BUNNY_API_KEY}
}
```
This configuration sets up the provider to use the Bunny DNS module with the API key provided as an environment variable. It ensures that your Caddy server can automatically issue and renew SSL certificates using DNS-01 challenges via Bunny.

This setup is the same as specifying the provider in the [tls directive's ACME issuer](https://caddyserver.com/docs/caddyfile/directives/tls#acme) configuration.

[Sample Caddyfile](#sample-caddyfile)

#### Troubleshooting

> [!NOTE]
> TODO: Haven't run into any issues yet to troubleshoot :)

[Official Bunny caddy-dns module repo for more information](https://github.com/caddy-dns/bunny)

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

## Platform Support

The `caddybuilds/caddy-bunny` image is built to support multiple platforms, ensuring compatibility across a wide range of devices and systems. The supported platforms include:

- **linux/amd64**: Standard x86_64 architecture, commonly used in desktop and server environments.
- **linux/arm64**: ARM 64-bit architecture, used in many modern servers and high-end ARM devices.
- **linux/arm/v7**: ARM 32-bit architecture, widely used in devices like Raspberry Pi.
- **linux/ppc64le**: 64-bit PowerPC Little Endian architecture.
- **linux/s390x**: 64-bit IBM System z architecture.

### Alpine

Both images are built on [Alpine Linux](https://alpinelinux.org/): the default through the official Caddy image, slim through caddy-base. The `alpine` and `<version>-alpine` tags are aliases of the default tags and resolve to the same digest.

### Raspberry Pi Support

This Docker image is optimized for Raspberry Pi, allowing you to deploy Caddy with Bunny DNS integration on these popular single-board computers. Whether you are using a Raspberry Pi 3 or the latest Raspberry Pi 4, this image provides the necessary support for seamless operation.

To use the image on a Raspberry Pi, ensure you are running a compatible operating system (such as Raspberry Pi OS) and have Docker installed. You can then pull the image and run it as you would on any other system:

```sh
docker pull ghcr.io/caddybuilds/caddy-bunny:latest
```
# How It Works

The pipeline is shared with the other CaddyBuilds images through [caddy-base](https://github.com/CaddyBuilds/caddy-base):

1. **Daily release check**: a scheduled workflow looks for new [Caddy releases](https://github.com/caddyserver/caddy/releases) and waits until the official image has every required platform.
2. **Module update detection**: the latest commit of each module in the `Dockerfile` is tracked in `module-versions.json`; new commits trigger a rebuild.
3. **Base image tracking**: if the official image is republished for base OS or Go fixes, the image is rebuilt on top of it.
4. **Multi-platform build**: the image is built for all supported architectures and pushed to GitHub Container Registry and, optionally, Docker Hub. Each release lists why it was built or rebuilt.
5. **Failure notifications**: a failed build opens a GitHub issue.

# Building Your Own Docker Image
If you prefer to build your own Docker image, follow these steps:

## Prerequisites

- GitHub account
- DockerHub account (if you wish to push the image to DockerHub as well)
- GitHub Secrets configured:
  - `GITHUB_TOKEN` (automatically available in GitHub Actions)
  - `DOCKERHUB_USERNAME` (optional, if you want to push to DockerHub)
  - `DOCKERHUB_TOKEN` (optional, if you want to push to DockerHub)
  - `DOCKERHUB_REPOSITORY_NAME` (optional, can be set as a repository secret or variable to customize the target DockerHub repository)


## Setup Instructions

1. **[Fork this repository](https://github.com/caddybuilds/caddy-bunny/fork)** to your GitHub account.

2. **Set up GitHub Secrets**:
   - Go to your repository on GitHub.
   - Navigate to `Settings` > `Secrets and variables` > `Actions`.
   - Add the following secrets:
     - `GITHUB_TOKEN`: This is automatically available in GitHub Actions.
     - `DOCKERHUB_USERNAME`: Your DockerHub username (optional).
     - `DOCKERHUB_TOKEN`: Your DockerHub access token (optional).
     - `DOCKERHUB_REPOSITORY_NAME` (optional, can be set as a repository secret or variable to customize the target DockerHub repository)

3. **Customize the image** (optional):
   - Add or change modules in the `--with` lines of the `Dockerfile`. The daily check reads the module list from there.
   - `caddy-image.json` holds the image title, description and the module ID verified after each build.
   - The workflows in `.github/workflows/` call the shared [caddy-base](https://github.com/CaddyBuilds/caddy-base) workflows, which handle the release check, the multi-platform build, tagging and releases.

4. **Manually trigger the workflow** (optional):
   - Go to the `Actions` tab in your GitHub repository.
   - Select the `Build and Push Docker Image` workflow.
   - Click the `Run workflow` button to trigger the build process manually.

5. **Enable the workflow**:
    - Make sure GitHub Actions is enabled for your repository. Go to the `Actions` tab and, if prompted, click "I understand my workflows, go ahead and enable them" to activate Actions for your fork.

6. **Monitor the workflow**:
   - You can monitor the progress and logs of the workflow in the `Actions` tab of your GitHub repository.

7. **Docker image**:
   - Once the workflow completes successfully, the Docker image will be available in the GitHub Container Registry under your repository.
   - You can pull the image using:
     ```sh
     docker pull ghcr.io/YOUR_GITHUB_USERNAME/YOUR_GITHUB_REPOSITORY_NAME:latest
     ```

## Usage

You can use the built Docker image in your projects. Here is an example of how to use it in a `docker-compose.yml` file:

```yaml
services:
  caddy:
    image: ghcr.io/YOUR_GITHUB_USERNAME/YOUR_GITHUB_REPOSITORY_NAME:latest
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
Defining the data volume as [external](https://docs.docker.com/compose/compose-file/compose-file-v3/#external) makes sure `docker-compose down` does not delete the volume. You may need to create it manually using `docker volume create caddy_data`.

Replace `YOUR_GITHUB_USERNAME` with your GitHub username and `your_bunny_api_key` with your actual Bunny API key.

## Contributing

Feel free to open issues or submit pull requests if you have any improvements or bug fixes.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
