# mbentley/omada-controller

Docker image for [TP-Link Omada Controller](https://www.tp-link.com/us/business-networking/omada-sdn-controller/) to control [TP-Link Omada Hardware](https://www.tp-link.com/en/business-networking/all-omada/)

For references on running a legacy v3 or v4 controller, see the [README for v3 and v4](README_v3_and_v4.md).

## Table of Contents

* [Image Tags](#image-tags)
    * [Multi-arch Tags](#multi-arch-tags)
    * [Explicit Architecture Tags](#explicit-architecture-tags)
    * [Archived Tags](#archived-tags)
* [Getting Help & Reporting Issues](#getting-help--reporting-issues)
* [Controller Upgrades](#controller-upgrades)
* [Building Images](#building-images)
* [Example Usage](#example-usage)
    * [Using non-default ports](#using-non-default-ports)
    * [Using port mapping](#using-port-mapping)
    * [Using `net=host`](#using-nethost)
* [Optional Variables](#optional-variables)
* [Persistent Data](#persistent-data-and-permissions)
* [Custom Certificates](#custom-certificates)
* [MongoDB Small Files](#mongodb-small-files)
* [Time Zones](#time-zones)
* [Unprivileged Ports](#unprivileged-ports)
* [Using Docker Compose](#using-docker-compose)
* [Omada Controller API Documentation](#omada-controller-api-documentation)
* [Known Issues](#known-issues)
    * [Containerization Issues](KNOWN_ISSUES.md#containerization-issues)
        * [MongoDB Corruption](KNOWN_ISSUES.md#mongodb-corruption)
        * [Notes for `armv7l`](KNOWN_ISSUES.md#notes-for-armv7l)
            * [:warning: Unsupported Base Image for `armv7l`](KNOWN_ISSUES.md#unsupported-base-image-for-armv7l)
            * [:warning: Unsupported MongoDB](KNOWN_ISSUES.md#unsupported-mongodb)
        * [Low Resource Systems](KNOWN_ISSUES.md#low-resource-systems)
        * [Mismatched Userland and Kernel](KNOWN_ISSUES.md#mismatched-userland-and-kernel)
    * [Upgrade Issues](KNOWN_ISSUES.md#upgrade-issues)
        * [5.8 - 404s and Blank Pages](KNOWN_ISSUES.md#58---404s-and-blank-pages)
        * [Incorrect CMD](KNOWN_ISSUES.md#incorrect-cmd)
        * [5.12 - Unable to Login After Upgrade](KNOWN_ISSUES.md#512---unable-to-login-after-upgrade)
        * [Slowness in Safari](KNOWN_ISSUES.md#slowness-in-safari)

## Image Tags

:warning: **Warning** :warning: Do **NOT** run the `armv7l` (32 bit) images. Upgrade your operating system to `arm64` (64 bit) unless you accept that you're running an outdated MongoDB and a base operating system with unpatched vulnerabilities! See the [Known Issues readme](KNOWN_ISSUES.md#notes-for-armv7l) for more information.

### Multi-arch Tags

The following tags have multi-arch support for `amd64`, `armv7l`, and `arm64` and will automatically pull the correct tag based on your system's architecture:

| Tag(s) | Major.Minor Release | Current Version |
| :----- | ------------------- | --------------- |
| `latest`, `5.13` | Omada Controller `5.13.x` | `5.13.30.8` |
| `beta` | Omada Controller `beta` | `5.14.20.9` |
| `5.12` | Omada Controller `5.12.x` | `5.12.7` |
| `5.9` | Omada Controller `5.9.x` | `5.9.31` |

### Tags with Chromium

**Note**: These are currently published for the `amd64` architecture only. These tags extend the tags above to add Chromium which is required to generate reports from the controller.

| Tag(s) | Major.Minor Release |
| :----- | ------------------- |
| `latest-chromium`, `5.13-chromium` | Omada Controller `5.13.x` |
| `beta-chromium`, | Omada Controller `beta` |
| `5.12-chromium` | Omada Controller `5.12.x` |
| `5.9-chromium` | Omada Controller `5.9.x` |

### Explicit Architecture Tags

See [list of architecture specific tags](ARCH_TAGS.md).

## Archived Tags

These images are still published on Docker Hub but are no longer regularly updated due to the controller software no longer being updated. **Use with extreme caution as these images are likely to contain unpatched security vulnerabilities!**

| Tag(s) | Major.Minor Release | Current Version |
| :----- | ------------------- | ----------------|
| `5.8` | Omada Controller `5.8.x` | `5.8.4` |
| `5.8-chromium` | Omada Controller `5.8.x` | `5.8.4` |
| `5.7` | Omada Controller `5.7.x` | `5.7.4` |
| `5.7-chromium` | Omada Controller `5.7.x` | `5.7.4` |
| `5.6` | Omada Controller `5.6.x` | `5.6.3` |
| `5.6-chromium` | Omada Controller `5.6.x` | `5.6.3` |
| `5.5` | Omada Controller `5.5.x` | `5.5.6` |
| `5.5-chromium` | Omada Controller `5.5.x` | `5.5.6` |
| `5.4` | Omada Controller `5.4.x` | `5.4.6` |
| `5.4-chromium` | Omada Controller `5.4.x` | `5.4.6` |
| `5.3` | Omada Controller `5.3.x` | `5.3.1` |
| `5.3-chromium` | Omada Controller `5.3.x` | `5.3.1` |
| `5.1` | Omada Controller `5.1.x` | `5.1.7` |
| `5.1-chromium` | Omada Controller `5.1.x` | `5.1.7` |
| `5.0` | Omada Controller `5.0.x` | `5.0.30` |
| `4.3` | Omada Controller `4.3.x` | `4.3.5` |
| `4.2` | Omada Controller `4.2.x` | `4.2.11` |
| `3.1` | Omada Controller `3.1.x` | `3.1.13` |
| `3.0` | Omada Controller `3.0.x` | `3.0.5` |

## Getting Help & Reporting Issues

If you have issues running the controller, feel free to [create a Help discussion](https://github.com/mbentley/docker-omada-controller/discussions/categories/help) and I will help as I can. If you are specifically having a problem that is related to the actual software, I would suggest filing an issue on the [TP-Link community forums](https://community.tp-link.com/en/business/forum/582) as I do not have access to source code to debug those issues. If you're not sure where the problem might be, I can help determine if it is a running in Docker issue or a software issue. If you're certain you have found a bug, create a [Bug Report Issue](https://github.com/mbentley/docker-omada-controller/issues/new/choose).

## Controller Upgrades

Controller upgrades are done by stopping the existing container gracefully (see the [note below](#preventing-database-corruption) on this topic), removing the existing container, and running a new container with the new version of the controller. This can be done manually, with compose, or with manby other 3rd party tools which auto-update containers.

### Preventing database corruption

When stopping your container in order to upgrade the controller, make sure to allow the MongoDB enough time to safely shutdown. This is done using `docker stop -t <value>` where `<value>` is a number in seconds, such as 60, which should allow the controller to cleanly shutdown. Database corruption has been observed when not cleanly shut down. The compose example now includes a default `stop_grace_period` of 60s.

## Building images

<details>
<summary>Click to expand docker build instructions</summary>

There are some differences between the build steps for `amd64`, `arm64`, and `armv7l`. These changes will happen automatically if you use the build-args `INSTALL_VER` and `ARCH`. For possible `INSTALL_VER` values, see [mbentley/docker-omada-controller-url](https://github.com/mbentley/docker-omada-controller-url/blob/master/omada_ver_to_url.sh):

### `amd64`

  No build args required; set for the default build-args

  ```
  docker build \
    --build-arg INSTALL_VER="5.13.30.8" \
    --build-arg ARCH="amd64" \
    -f Dockerfile.v5.x \
    -t mbentley/omada-controller:5.13 .
  ```

### `arm64`

  Only the `ARCH` build-arg is required

  ```
  docker build \
    --build-arg INSTALL_VER="5.13.30.8" \
    --build-arg ARCH="arm64" \
    -f Dockerfile.v5.x \
    -t mbentley/omada-controller:5.13-arm64 .
  ```

### `armv7l`

  Both the `ARCH` and `BASE` build-args are required

  ```
  docker build \
    --build-arg INSTALL_VER="5.13.30.8" \
    --build-arg ARCH="armv7l" \
    --build-arg BASE="ubuntu:16.04" \
    -f Dockerfile.v5.x \
    -t mbentley/omada-controller:5.13-armv7l .
  ```

</details>

## Example Usage

To run this Docker image and keep persistent data in named volumes:

### Using non-default ports

__tl;dr__: Always make sure the environment variables for the ports match any changes you have made in the web UI and you'll be fine.

If you want to change the ports of your Omada Controller to something besides the defaults, there is some unexpected behavior that the controller exhibits. There are two sets of ports: one for HTTP/HTTPS for the controller itself and another for HTTP/HTTPS for the captive portal, typically used for authentication to a guest network. The controller's set of ports, which are set by the `MANAGE_*_PORT` environment variables, can only be modified using the environment variables on the first time the controller is started. If persistent data exists, changing the controller's ports via environment variables will have no effect on the controller itself and can only be modified through the web UI. On the other hand, the portal ports will always be set to whatever has been set in the environment variables, which are set by the `PORTAL_*_PORT` environment variables.

### Using port mapping

__Warning__: If you want to change the controller ports from the default mappings, you *absolutely must* update the port binding inside the container via the environment variables. The ports exposed must match what is inside the container. The Omada Controller software expects that the ports are the same inside the container and outside and will load a blank page if that is not done. See [#99](https://github.com/mbentley/docker-omada-controller/issues/99#issuecomment-821243857) for details and and example of the behavior.

```bash
docker run -d \
  --name omada-controller \
  --stop-timeout 60 \
  --restart unless-stopped \
  --ulimit nofile=4096:8192 \
  -p 8088:8088 \
  -p 8043:8043 \
  -p 8843:8843 \
  -p 27001:27001/udp \
  -p 29810:29810/udp \
  -p 29811-29816:29811-29816 \
  -e MANAGE_HTTP_PORT=8088 \
  -e MANAGE_HTTPS_PORT=8043 \
  -e PGID="508" \
  -e PORTAL_HTTP_PORT=8088 \
  -e PORTAL_HTTPS_PORT=8843 \
  -e PORT_ADOPT_V1=29812 \
  -e PORT_APP_DISCOVERY=27001 \
  -e PORT_DISCOVERY=29810 \
  -e PORT_MANAGER_V1=29811 \
  -e PORT_MANAGER_V2=29814 \
  -e PORT_TRANSFER_V2=29815 \
  -e PORT_RTTY=29816 \
  -e PORT_UPGRADE_V1=29813 \
  -e PUID="508" \
  -e SHOW_SERVER_LOGS=true \
  -e SHOW_MONGODB_LOGS=false \
  -e SSL_CERT_NAME="tls.crt" \
  -e SSL_KEY_NAME="tls.key" \
  -e TZ=Etc/UTC \
  -v omada-data:/opt/tplink/EAPController/data \
  -v omada-logs:/opt/tplink/EAPController/logs \
  mbentley/omada-controller:5.13
```

### Using `net=host`

In order to use the host's network namespace, you must first ensure that there are not any port conflicts. The `docker run` command is the same except for that all of the published ports should be removed and `--net host` should be added. Technically it will still work if you have the ports included, but Docker will just silently drop them. Here is a snippet of what the above should be modified to look like:

```bash
...
  --restart unless-stopped \
  --net host \
  -e MANAGE_HTTP_PORT=8088 \
...
```

## Optional Variables

| Variable | Default | Values | Description | Valid For |
| :------- | :------ | :----: | :---------- | :-------: |
| `MANAGE_HTTP_PORT` | `8088` | `1024`-`65535` | Management portal HTTP port; for ports < 1024, see [Unprivileged Ports](#unprivileged-ports) | >= `3.2` |
| `MANAGE_HTTPS_PORT` | `8043` | `1024`-`65535` | Management portal HTTPS port; for ports < 1024, see [Unprivileged Ports](#unprivileged-ports) | >= `3.2` |
| `PGID` | `508` | _any_ | Set the `omada` process group ID ` | >= `3.2` |
| `PGROUP` | `omada` | _any_ | Set the group name for the process group ID to run as | >= `5.0` |
| `PORTAL_HTTP_PORT` | `8088` | `1024`-`65535` | User portal HTTP port; for ports < 1024, see [Unprivileged Ports](#unprivileged-ports) | >= `4.1` |
| `PORTAL_HTTPS_PORT` | `8843` | `1024`-`65535` | User portal HTTPS port; for ports < 1024, see [Unprivileged Ports](#unprivileged-ports) | >= `4.1` |
| `PORT_ADOPT_V1` | `29812` | `1024`-`65535` | Omada Controller and Omada Discovery Utility manage the Omada devices running firmware fully adapted to Omada Controller v4* | >= `5.x` |
| `PORT_APP_DISCOVERY` | `27001` | `1024`-`65535` | Omada Controller can be discovered by the Omada APP within the same network through this port | >= `5.x` |
| `PORT_DISCOVERY` | `29810` | `1024`-`65535` | Omada Controller and Omada Discovery Utility discover Omada devices | >= `5.x` |
| `PORT_MANAGER_V1` | `29811` | `1024`-`65535` | Omada Controller and Omada Discovery Utility manage the Omada devices running firmware fully adapted to Omada Controller v4* | >= `5.x` |
| `PORT_MANAGER_V2` | `29814` | `1024`-`65535` | Omada Controller and Omada Discovery Utility manage the Omada devices running firmware fully adapted to Omada Controller v5* | >= `5.x` |
| `PORT_TRANSFER_V2` | `29815` | `1024`-`65535` | Omada Controller receives Device Info and Packet Capture files from the Omada devices | >= `5.9` |
| `PORT_RTTY` | `29816` | `1024`-`65535` | Omada Controller establishes the remote control terminal session with the Omada devices | >= `5.9` |
| `PORT_UPGRADE_V1` | `29813` | `1024`-`65535` | When upgrading the firmware for the Omada devices running firmware fully adapted to Omada Controller v4*. | >= `5.x` |
| `PUID` | `508` | _any_ | Set the `omada` process user ID ` | >= `3.2` |
| `PUSERNAME` | `omada` | _any_ | Set the username for the process user ID to run as | >= `5.0` |
| `SHOW_SERVER_LOGS` | `true` | `[true\|false]` | Outputs Omada Controller logs to STDOUT at runtime | >= `4.1` |
| `SHOW_MONGODB_LOGS` | `false` | `[true\|false]` | Outputs MongoDB logs to STDOUT at runtime | >= `4.1` |
| `SKIP_USERLAND_KERNEL_CHECK` | `false` | `[true\|false]` | When set to `true`, skips the userland/kernel match check for `armv7l` & `arm64` |
| `SMALL_FILES` | `false` | `[true\|false]` | See [Small Files](#small-files) for more detail; no effect in >= `4.1.x` | `3.2` only |
| `SSL_CERT_NAME` | `tls.crt` | _any_ | Name of the public cert chain mounted to `/cert`; see [Custom Certificates](#custom-certificates) | >= `3.2` |
| `SSL_KEY_NAME` | `tls.key` | _any_ | Name of the private cert mounted to `/cert`; see [Custom Certificates](#custom-certificates) | >= `3.2` |
| `TLS_1_11_ENABLED` | `false` | `[true\|false]` | Re-enables TLS 1.0 & 1.1 if set to `true` | >= `4.1` |
| `TZ` | `Etc/UTC` | _\<many\>_ | See [Time Zones](#time-zones) for more detail | >= `3.2` |

Documentation on the ports used by the controller can be found in the [TP-Link FAQ](https://www.tp-link.com/us/support/faq/3281/).

## Persistent Data

In the examples, there are two directories where persistent data is stored: `data` and `logs`. The `data` directory is where the persistent database data is stored where all of your settings, app configuration, etc is stored. The `log` directory is where logs are written and stored. I would suggest that you use a bind mounted volume for the `data` directory to ensure that your persistent data is directly under your control and of course take regular backups within the Omada Controller application itself.

## Custom Certificates

By default, Omada software uses self-signed certificates. If however you want to use custom certificates you can mount them into the container as `/cert/tls.key` and `/cert/tls.crt`. The `tls.crt` file needs to include the full chain of certificates, i.e. cert, intermediate cert(s) and CA cert. This is compatible with kubernetes TLS secrets. Entrypoint script will convert them into Java Keystore used by jetty inside the Omada SW. If you need to use different file names, you can customize them by passing values for `SSL_CERT_NAME` and `SSL_KEY_NAME` as seen above in the [Optional Variables](#optional-variables) section.

**Warning** - As of the version 4.1, certificates can also be installed through the web UI. You should not attempt to mix certificate management methods as installing certificates via the UI will store the certificates in MongoDB and then the `/cert` volume method will cease to function.

## Time Zones

By default, this image uses the `Etc/UTC` time zone. You may update the time zone used by passing a different value in the `TZ` variable. See [List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List) for a complete list of values in the `TZ database name` table column.

## Unprivileged Ports

This Docker image runs as a non-root user by default. In order to bind unprivileged ports (ports < 1024 by default), you must include `--sysctl net.ipv4.ip_unprivileged_port_start=0` in your `docker run` command to allow ports below 1024 to be bound by non-root users.

## Using Docker Compose

There is a [Docker Compose file](https://github.com/mbentley/docker-omada-controller/blob/master/docker-compose.yml) available for those who would like to use compose to manage the lifecycle of their container:

```bash
wget https://raw.githubusercontent.com/mbentley/docker-omada-controller/master/docker-compose.yml
docker compose up -d
```

## Omada Controller API Documentation

If you are interested in using the Omada Controller APIs to retrieve data from the controller, the latest version of the API documentation that I have found is available from the [community forums in this post](https://community.tp-link.com/en/business/forum/topic/590430). I'm not able to provide support for the APIs but I've found them to be helpful for my own usage and they weren't easy to find.

## Known Issues

See [the Known Issues](KNOWN_ISSUES.md) documentation for details.