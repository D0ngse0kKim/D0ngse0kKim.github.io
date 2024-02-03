---
layout: distill
title: Docker Storage
description: Docker Storage
# img: assets/img/12.jpg
# importance: 1
category: Docker

toc:
  - name: Abstract
  - name: Volume
  - name: Bind Mounts
  - name: tmpfs Mounts
---

## Abstract

There is three types of storage in docker:

Volume
: A volume is a directory that is stored outside of the container’s
filesystem. It is often used for persistent data.

Bind Mounts
: A bind mounts is a file or directory stored anywhere on the
container host filesystem, mounted into a running container.

tmpfs mounts
: A tmpfs mounts is stored in the host system’s memory only, and
is never written to the host system’s filesystem.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {%
        include figure.html path="tools/docker/storage/types-of-mounts.webp"
        class="img-fluid rounded z-depth-1" zoomable=true
        caption="Types of mounts"
        %}
    </div>
</div>

## Volume

Details can be found in [this page](https://docs.docker.com/storage/volumes/).

### Create and manage volumes

Before using a volume, it must be created. There are several commands to manage
volumes.

#### Create a volume

```bash
$ docker volume create my-volume
```

See [docker volume create](https://docs.docker.com/engine/reference/commandline/volume_create/).

#### List volumes

```bash
$ docker volume ls
```

See [docker volume ls](https://docs.docker.com/engine/reference/commandline/volume_ls/).

#### Inspect a volume

```bash
$ docker volume inspect my-volume
```

See [docker volume inspect](https://docs.docker.com/engine/reference/commandline/volume_inspect/).

#### Remove a volume

```bash
$ docker volume rm my-volume
```

See [docker volume rm](https://docs.docker.com/engine/reference/commandline/volume_rm/).

### Mount a volume to a container

```bash
$ docker run -d \
--name my-container \
--mount type=volume,source=my-volume,destination=/mnt/my-volume \
ubuntu:latest
```

#### `--mount` options

The `--mount` option is used to mount a volume to a container. It accepts
key-value pairs separated by commas. The following options are available:

`type`
: The type of the mount, which can be `volume`, `bind`, or `tmpfs`. When
mounting a volume, the `type` must be `volume`.

`source`
: The name of the volume to mount. When mounting a volume, the `source` is
the name of the volume to mount. Can be specified as `source` or `src`.

`destination`
: The path in the container where the volume is mounted. Can be specified as
`destination`, `dst`, or `target`.

`readonly`
: Mount the volume as read-only. Can be specified as `readonly` or `ro`.
There is no need to specify a value with this option by suffixing
`=value` parts.

`volume-opt`
: Options to pass to the volume driver which is specified in
the [`--opt` option of `docker volume create` command](https://docs.docker.com/engine/reference/commandline/volume_create/#opt).
Can be specified multiple times.

## Bind Mounts

Details can be found in
[this page](https://docs.docker.com/storage/bind-mounts/).

Bind mounts does not need to be created before using it. It can be created
on the fly.

### Mount a bind mount to a container

```bash
$ docker run -d \
--name my-container \
--mount type=bind,source="$(pwd)/src/in/host",destination=/mnt/my-bind-mount \
ubuntu:latest
```

#### `--mount` options

The `--mount` option is used to mount a bind mount to a container. It accepts
key-value pairs separated by commas. The following options are available:

`type`
: The type of the mount, which can be `volume`, `bind`, or `tmpfs`. When
mounting a bind mount, the `type` must be `bind`.

`source`
: The path of the bind mount on the host. When mounting a bind mount, the
`source` is the path of the bind mount on the host. Can be specified as
`source` or `src`.

`destination`
: The path in the container where the bind mount is mounted. Can be specified
as `destination`, `dst`, or `target`.

`readonly`
: Mount the bind mount as read-only. Can be specified as `readonly` or `ro`.
There is no need to specify a value with this option by suffixing
`=value` parts.

[`bind-propagation`](https://docs.docker.com/storage/bind-mounts/#configure-bind-propagation)
: The propagation mode of the bind mount. Can be `rprivate`, `private`,
`rshared`, `shared`, `rslave`, or `slave`.

[`bind-recursive`](https://docs.docker.com/storage/bind-mounts/#recursive-mounts)
: Mount the submounts of the bind mount recursively.

## tmpfs Mounts

Details can be found in [this page](https://docs.docker.com/storage/tmpfs/).

tmpfs cannot be created before using it. It can be created on the fly.

### Mount a tmpfs mount to a container

```bash
$ docker run -d \
--name my-container \
--mount type=tmpfs,destination=/mnt/my-tmpfs \
ubuntu:latest
```

#### `--mount` options

The `--mount` option is used to mount a tmpfs mount to a container. It accepts
key-value pairs separated by commas. The following options are available:

`type`
: The type of the mount, which can be `volume`, `bind`, or `tmpfs`. When
mounting a tmpfs mount, the `type` must be `tmpfs`.

`destination`
: The path in the container where the tmpfs mount is mounted. Can be specified
as `destination`, `dst`, or `target`.

`tmpfs-size`, `tmpfs-mode`
: The size and file mode of the tmpfs mount.
See [tmpfs options](https://docs.docker.com/storage/tmpfs/#specify-tmpfs-options).

## Docker Compose

### Example

The following example sets three volumes to a container.

* `my-volume-name` : A volume named `my-volume-name` is created and attached
    to the container.
* `my-volume-name-2` : A volume named `my-volume-name-2` is created and
    attached to the container.
* `my-volume-name-3` : A bind mount is attached to the container.

```yaml
services:
  my-service-name:
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=secret
    volumes:
      - my-volume-name:/var/lib/mysql
      - type: volume
        source: my-volume-name-2
        target: /var/lib/mysql2
        volume:
          nocopy: true
      - type: bind
        source: .
        target: /var/lib/mysql3
        volume:
          nocopy: true

volumes:
  my-volume-name:
  my-volume-name-2:
```

When using volumes in Docker Compose, definition and attachment of volumes
consists of two parts:

1. [`volumes` element in `services` top-level element](https://docs.docker.com/compose/compose-file/05-services/#volumes)
    * `volumes` element in `services` top-level element defines volumes to be
      attached to containers.
2. [`volumes` top-level element](https://docs.docker.com/compose/compose-file/07-volumes/)
    * `volumes` top-level element defines volumes to be created.

### `volumes` element in `services` top-level element

This part defines how the volumes will be attached to containers.

`type`
: The type of the mount, which can be `volume`, `bind`, or `tmpfs`, `npip` or
`cluster`.

`source`
: The name of the volume to mount.

1. When mounting a volume, the `source` is the name of the volume to mount.
2. When mounting a bind mount, the `source` is the path of the bind mount on
    the host.
3. When mounting a tmpfs mount, the `source` is not used.

`target`
: The path in the container where the volume is mounted.

`read_only`
: Mount the volume as read-only.

`bind`
: Additional options for bind mounts.

1. `propagation` : The propagation mode of the bind mount.
    * Can be `rprivate`, `private`, `rshared`, `shared`, `rslave`, or `slave`.
2. `create_host_path` : Create the path on the host if it does not exist.
    Compose does nothing if there is something present at the path.
3. `selinux` : The SELinux context of the bind mount.
    * Can be `z` (shared), `Z` (private)

`volume`
: Additional options for volumes.

1. `nocopy` : Do not copy data from the container before removing it.
    * Can be `true` or `false`.
    * Default is `false`.

`tmpfs`
: Additional options for tmpfs mounts.

1. `size` : The size of the tmpfs mount in bytes (numeric or as bytes unit).
2. `mode` : The file mode of the tmpfs mount as Unix permission bits as
    an octal number.

`consistency`
: The consistency requirements of the mount. Available values are platform
specific.

### `volumes` top-level element

This element defines how the volumes will be created.

The second-level elements are the names of the volumes to be created. The
values of the elements are the options for the volumes.

Second-level elements
: The names of the volumes to be created automatically. The automatically
created volumes are named as `projectname_volumename`.

`driver`
: The name of the volume driver to use.
In [this page](https://docs.docker.com/storage/volumes/#use-a-volume-driver),
volume driver `vieux/sshfs` is used. In this case, the value of `driver` is
`vieux/sshfs`.

```yaml
volumes:
  my-volume-name:
    driver: vieux/sshfs
```

`driver_opts`
: Options to pass to the volume driver. In the examples of `vieux/sshfs`,
the following [driver-specific options](https://docs.docker.com/engine/reference/commandline/volume_create/#opt)
are used:

```bash
docker volume create --driver vieux/sshfs \
  -o sshcmd=test@node2:/home/test \
  -o password=testpassword \
  sshvolume
```

This is equivalent to the following:

```yaml
volumes:
  my-volume-name:
    driver: vieux/sshfs
    driver_opts:
      sshcmd: test@node2:/home/test
      password: testpassword
```

`external`
: Determines whether or not the volume is created outside of the Compose file.

1. `true`
    * The volume already exists on the host machine
    * Its lifecycle is managed outside of the Compose file.
    * The volume will never be removed by Compose.
    * All other attributes except `name` are ignored.
2. `false`
    * The default value of `external`.

In the example below, the volume `db-data` is created outside of the Compose
file instead of being created a volume named `{project_name}_db-data`.

```yaml
services:
  backend:
    image: example/database
    volumes:
      - db-data:/etc/data

volumes:
  db-data:
    external: true
```

`name`
: Custom name for the volume. If not set, Compose uses the volume’s name as
defined by the `volumes` section of the Compose file.

```yaml
volumes:
  db-data:
    name: "my-app-data"
```

`labels`
: Labels to apply to the volume. An array or a dictionary can be used.

```yaml
volumes:
  db-data:
    labels:
      com.example.description: "Database volume"
      com.example.department: "IT/Ops"
      com.example.label-with-empty-value: ""
```

```yaml
volumes:
  db-data:
    labels:
      - "com.example.description=Database volume"
      - "com.example.department=IT/Ops"
      - "com.example.label-with-empty-value"
```
