# Images

This article walks you through how to
[configure images](../../../images/configure.md) for use with CVMs, as well as
how to access images located in private [registries](../../registries/index.md).

## Image configuration

The following sections show how you can configure your image to include systemd
and Docker for use in CVMs.

### systemd

If your image's OS distribution doesn't link the `systemd` init to `/sbin/init`,
you'll need to do this manually in your Dockerfile.

The following snippet shows how you can specify `systemd` as the init in your
image:

```Dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y \
    build-essential \
    systemd

# use systemd as the init
RUN ln -s /lib/systemd/systemd /sbin/init
```

When you start up a workspace, Coder checks for the presence of `/sbin/init` in
your image. If it exists, then Coder uses it as the container entrypoint with a
`PID` of 1.

### Docker

To add Docker, install the `docker` packages into your image. For a seamless
experience, use [systemd](#systemd) and register the `docker` service so
`dockerd` runs automatically during initialization.

The following snippet shows how your image can register the `docker` services in
its Dockerfile. For a full example, [see our `enterprise-base` image for a full example](https://github.com/coder/enterprise-images/blob/main/images/base/Dockerfile.ubuntu).

```Dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y \
    build-essential \
    git \
    bash \
    docker.io \
    curl \
    sudo \
    systemd

# Enables Docker starting with systemd
RUN systemctl enable docker

# use systemd as the init
RUN ln -s /lib/systemd/systemd /sbin/init

# Add docker-compose
RUN curl -L "https://github.com/docker/compose/releases/download/2.4.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose \
&& chmod +x /usr/local/bin/docker-compose

# Add a user `coder` so that you're not developing as the `root` user
RUN useradd coder \
      --create-home \
      --shell=/bin/bash \
      --groups=docker \
      --uid=1000 \
      --user-group && \
    echo "coder ALL=(ALL) NOPASSWD:ALL" >>/etc/sudoers.d/nopasswd

USER coder
```

## Private registries

To use CVM workspaces with private images, you **must** create a
[registry](../../registries/index.md#adding-a-registry) with authentication
credentials. Private images that can be pulled directly by the node will not
work with CVMs.

This restriction does not apply if you enable
[cached CVMs](../cvms/management.md#enabling-cached-cvms).
