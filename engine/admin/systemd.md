---
description: Controlling and configuring Docker using systemd
keywords: docker, daemon, systemd,  configuration
redirect_from:
- /engine/articles/systemd/
title: Control and configure Docker with systemd
---

Many Linux distributions use systemd to start the Docker daemon. This document
shows a few examples of how to customize Docker's settings.

## Start the Docker daemon

### Start manually

Once Docker is installed, you will need to start the Docker daemon.
Most Linux distributions use `systemctl` to start services. If you
do not have `systemctl`, use the `service` command.

- **`systemctl`**:

  ```bash
  $ sudo systemctl start docker
  ```

- **`service`**:
  
  ```bash
  $ sudo service docker start
  ```

### Start automatically at system boot

If you want Docker to start at boot, see
[Configure Docker to start on boot](/engine/installation/linux/linux-postinstall.md/#configure-docker-to-start-on-boot).

## Custom Docker daemon options

There are a number of ways to configure the daemon flags and environment variables
for your Docker daemon. The recommended way is to use the platform-independent
`daemon.json` file, which is located in `/etc/docker/` on Linux by default. See
[Daemon configuration file](/engine/reference/commandline/dockerd.md/#daemon-configuration-file).

You can configure nearly all daemon configuration options using `daemon.json`. The following
example configures two options. One thing you cannot configure using `daemon.json` mechanism is
a [HTTP proxy](#http-proxy).

### Runtime directory and storage driver

You may want to control the disk space used for Docker images, containers
and volumes by moving it to a separate partition.

To accomplish this, set the following flags in the `daemon.json` file:

```none
{
    "graph": "/mnt/docker-data",
    "storage-driver": "overlay"
}
```

### HTTP proxy

The Docker daemon uses the `HTTP_PROXY` and `NO_PROXY` environmental variables in
its start-up environment to configure HTTP proxy behavior. You cannot configure
these environment variables using the `daemon.json` file.

This example overrides the default `docker.service` file.

If you are behind an HTTP proxy server, for example in corporate settings,
you will need to add this configuration in the Docker systemd service file.

1.  Create a systemd drop-in directory for the docker service:

    ```bash
    $ mkdir -p /etc/systemd/system/docker.service.d
    ```

2.  Create a file called `/etc/systemd/system/docker.service.d/http-proxy.conf`
    that adds the `HTTP_PROXY` environment variable:

    ```conf
    [Service]
    Environment="HTTP_PROXY=http://proxy.example.com:80/"
    ```

3.  If you have internal Docker registries that you need to contact without
    proxying you can specify them via the `NO_PROXY` environment variable:

    ```conf
    Environment="HTTP_PROXY=http://proxy.example.com:80/" "NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com"
    ```

4.  Flush changes:

    ```bash
    $ sudo systemctl daemon-reload
    ```

5.  Restart Docker:

    ```bash
    $ sudo systemctl restart docker
    ```

6.  Verify that the configuration has been loaded:

    ```bash
    $ systemctl show --property=Environment docker
    Environment=HTTP_PROXY=http://proxy.example.com:80/
    ```

## Manually creating the systemd unit files

When installing the binary without a package, you may want
to integrate Docker with systemd. For this, install the two unit files
(`service` and `socket`) from [the github
repository](https://github.com/docker/docker/tree/master/contrib/init/systemd)
to `/etc/systemd/system`.
