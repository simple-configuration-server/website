---
layout: default
title: "Docker"
description: "Use the official docker image to deploy SCS"
permalink: "/docs/deployment/docker"
nav_order: 1
grand_parent: Documentation
parent: Deployment
---
# Docker Deployment
The official SCS Docker image is published in the
[SCS GitHub container registry](https://github.com/simple-configuration-server/simple-configuration-server/pkgs/container/simple-configuration-server):
```
ghcr.io/simple-configuration-server/simple-configuration-server:$VERSION
```

There are two ways to deploy SCS using Docker:
1. Use the Docker image directly, and use Docker bind-mounts to expose the
   relevant folders and files to the Docker container.
2. Build your own SCS Docker image using the official SCS image and include
   your configuration files, with the exception of secrets.

The first option is the quickest way to deploy SCS from scratch. Below is an
example Docker compose-file for a Docker service using the official Docker
image:
```yaml
version: "3.9"

services:
  server:
    image: ghcr.io/simple-configuration-server/simple-configuration-server:1.0.3
    volumes:
      # If all configuration, except secrets, are in a repository:
      - ./configuration-repository/configuration:/etc/scs/configuration
      # The secrets are mounted to a seperate directory. Set directories.secrets
      # in scs-configuration.yaml accordingly
      - ./.local/secrets:/etc/scs/secrets
      # When SCS_DISABLE_SSL=0 (default) you need to mount your cert (chain) and
      # private key
      - ./.local/ssl/cert-chain.crt:/etc/ssl/certs/scs.crt
      - ./.local/ssl/private-key.key:/etc/ssl/private/scs.key
      # Use a docker-volume for the logs (In case you use e.g. Filebeat, as
      # Described on the 'Log Centralization' page)
      - scs-logs:/var/log/scs
    environment:
      - SCS_CONFIG_DIR=/etc/scs/configuration
      # The following 2 variables can be omitted if the default values are
      # used.
      #
      # It's recommended to only disable SSL when using a reverse proxy,
      # like NGINX, for SSL termination
      - SCS_DISABLE_SSL=0
      #
      # Set the reverse proxy count >= 1 if you're using a reverse proxy that
      # sets the X-Forwarded-For header. This means the 'X-Forwarded-For'
      # header is used as the Remote Address, rather than the IP of the proxy
      # itself. The count indicates how many values should be in the
      # X-Forwarded-For headers. If you use multiple proxies, this may be
      # higher than one.
      - SCS_REVERSE_PROXY_COUNT=0
    ports:
      - 3000:80
    restart: on-failure:5
```

The second option takes more work to set-up, but enables you to version control
your complete configuration, including SCS version and possible extensions.
An example of using Gitlab CI/CD or Github Actions to build a custom
Docker image can be found in the
[example-configuration](https://gitlab.com/simple-configuration-server/example-configuration)
repository.
