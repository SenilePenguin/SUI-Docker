**SUI-docker**

**A startpage for your server and / or new tab page**

---

Creates a Container which runs [SenilePenguin's](https://github.com/senilepenguin) [sui](https://github.com/senilepenguin/sui), with [lsiobase/nginx](https://hub.docker.com/r/lsiobase/nginx) as the base image, as seen on https://imgur.com/a/FDVRIyw.

The [lsiobase/nginx](https://hub.docker.com/r/lsiobase/nginx) image is a custom base image built with [Alpine linux](https://alpinelinux.org/) and [S6 overlay](https://github.com/just-containers/s6-overlay).
Using this image allows us to use the same user/group ids in the container as on the host, making file transfers much easier

# Deployment

Tags | Description
-----|------------
`latest` | Using the `latest` tag will pull the latest image for linux/amd64,linux/arm/v7,linux/arm64.
`develop` | The latest image of, if existent, the in-dev version of this container. Use at your own risk!

Using GitHub Workflows, images for this container are multi-arch. Simply pulling `:latest` should retrieve the correct image for your architecture.
Images are available for linux/amd64,linux/arm/v7,linux/arm64.

## Pre-built images

Using docker-compose:

```docker-compose.yml
version: "2"
services:
  sui:
    image: senilepenguin/sui:latest
    container_name: sui
    restart: unless-stopped
    environment:
      - TZ=America/New_York # Timezone
      - PUID=1000 # User ID
      - PROTOCOL=https # The protocol used to access this container. Either HTTP or HTTPS.
      - PGID=1000 # Group ID
      - DOMAIN=www.example.com # The address of the device this container is running on. Can be an IP or sub.domain.tld.
    volumes:
      - /host/path/to/config:/config # Contains all application data and base-image config files
    ports:
      - 443:443/tcp # https
      - 80:80/tcp # http
```

Using CLI:

```bash
docker create \
  --name=sui \
  -e TZ=America/New_York `# Timezone` \
  -e PUID=1000 `# User ID` \
  -e PROTOCOL=https `# The protocol used to access this container. Either HTTP or HTTPS.` \
  -e PGID=1000 `# Group ID` \
  -e DOMAIN=www.example.com `# The address of the device this container is running on. Can be an IP or sub.domain.tld.` \
  -v /host/path/to/config:/config `# Contains all application data and base-image config files` \
  -p 4433:443/tcp `# https` \
  -p 803:80/tcp `# http` \
  --restart unless-stopped \
  senilepenguin/sui:latest
```

# Configuration

Configuration | Explanation
------------ | -------------
[Restart policy](https://docs.docker.com/compose/compose-file/#restart) | "no", always, on-failure, unless-stopped
config volume | Contains config files and logs.
data volume | Contains your/the containers important data.
TZ | Timezone
PUID | for UserID
PGID | for GroupID
DOMAIN | The address of the device this container is running on. Can be an IP or sub.domain.tld.
PROTOCOL | The protocol used to access this container. Either HTTP or HTTPS.
ports | The port where the service will be available at.

## User / Group Identifiers

When using volumes, permissions issues can arise between the host OS and the container. [Linuxserver.io](https://www.linuxserver.io/) avoids this issue by allowing you to specify the user `PUID` and group `PGID`.

Ensure any volume directories on the host are owned by the same user you specify and any permissions issues will vanish like magic.

In this instance `PUID=1000` and `PGID=1000`, to find yours use `id user` as below:

```
  $ id username
    uid=1000(dockeruser) gid=1000(dockergroup) groups=1000(dockergroup)
```

# Building the image yourself

Use the [Dockerfile](https://github.com/senilepenguin/sui-docker/Dockerfile) to build the image yourself, in case you want to make any changes to it

Using docker-compose:

```docker-compose.yml
version: '3.6'
services:
  sui:
    container_name: sui
    build: ./sui-docker/
    restart: unless-stopped
    volumes:
      - ./path/to/config/files:/config
    environment:
      - TZ=America/New_York
      - PUID=1000  # User ID
      - PGID=1000  # Group ID
      - DOMAIN=www.example.com
      - PROTOCOL=https
    ports:
      - 803:80
      - 4433:443
```

1. Clone the repository: `git clone https://github.com/senilepenguin/sui-docker.git ./sui-docker`
1. Prepare docker-compose.yml file as seen above
1. `docker-compose up -d --build sui`
1. Visit IP.ADDRESS.OF.HOST:803
1. ???
1. Profit!
