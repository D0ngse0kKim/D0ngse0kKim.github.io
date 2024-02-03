---
layout: distill
title: Docker CLI Cheat Sheet
description: Docker CLI Cheat Sheet
# img: assets/img/12.jpg
# importance: 1
category: Docker

toc:
  - name: Image
  - name: docker build
  - name: Container
---


## Image

### [`docker build`](https://docs.docker.com/engine/reference/commandline/build/)

Build an image from a `Dockerfile`.

#### Synopsis

```bash
$ docker build [OPTIONS] PATH | URL | -
```

#### Example

```bash
$ docker build -t getting-started:v1 .
```

* `-t getting-started:v1` : Specify a name of the image(`getting-started`) and tag(`v1`)
* `.` : Specify a working folder which contains `Dockerfile`


### [`docker images`](https://docs.docker.com/engine/reference/commandline/images/)

List images

```bash
$ docker images
REPOSITORY        TAG       IMAGE ID       CREATED       SIZE
getting-started   latest    cf596f6f1748   8 hours ago   221MB
```

### [`docker image rm`](https://docs.docker.com/engine/reference/commandline/image_rm/)

Remove images.

* `--force`, `-f` : Force removal of the image

## Container

### [`docker run`](https://docs.docker.com/engine/reference/commandline/run/)

Create and run a new container from the image.

#### Synopsis

```bash
$ docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

#### `[OPTIONS]`

| Option                     | Short | Default | Description                                        |
| -------------------------- | ----- | ------- | -------------------------------------------------- |
| `--name`                   |       |         | Assign a name to the container.                    |
| `--publish host:container` | `-p`  |         | Publish a container's port(s) to the host          |
| `--rm`                     |       |         | Automatically remove the container when it exits   |
| `--detach`                 | `-d`  |         | Run container in background and print container ID |
| `--interactive`            | `-i`  |         | Keep STDIN open even if not attached               |
| `--tty`                    | `-t`  |         | Allocate a pseudo-TTY                              |
| `--mount`                  |       |         | Attach a filesystem mount to the container         |
| `--network`                |       |         | Connect a container to a network                   |
| `--network-alias`          |       |         | Add network-scoped alias for the container         |

* `-i` or `--interactive`: This flag keeps `STDIN` open even if not attached. This means that it allows you to interact with the container.
* `-t` or `--tty`: This flag allocates a pseudo-TTY. In simpler terms, it simulates a real terminal, like what you would get when you SSH into a machine.

#### `IMAGE`

Specify the name of image with tag `name:tag`. If tag is omitted,
the tag `latest` is specified.

#### `[COMMAND] [ARG...]`

Specify a command and argument when running the container.

#### Example

```bash
$ docker run --name getting-started-c -p 127.0.0.1:3000:3000 -d getting-started:v1
```

* `--name getting-started-c` : Assign a name `getting-started-c` to the container.
* `-p 127.0.0.1:3000:3000` : Publish a container's port `3000` to the host's port  
    `3000`.
* `-d` : Run container in background and print container ID
* `getting-started:v1` : Specify the name of image `getting-started` with tag `v1`.

#### Example

```bash
docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
```

* `-d` : Run container in background and print container ID
* `ubuntu` : Specify the name of image `ubuntu` with tag `latest`.
* `bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"` : Specify a command and argument when running the container.

#### `--mount` option

It is preferred to use `--mount` option instead of `-v` or `--volume` option.
`--mount` option consists of multiple key-value pairs with comma(`,`) separated.

| Key                | Description                                                                                                |
| ------------------ | ---------------------------------------------------------------------------------------------------------- |
| `type`             | The type of the mount, which can be `bind`, `volume`, or `tmpfs`.                                          |
| `source`           | The source of the mount. Depends on `type`.                                                                |
| `destination`      | The path where the file or directory is mounted in the container.                                          |
| `readonly`         | Mount the volume as read-only or not.                                                                      |
| `bind-propagation` | Changes the bind propagation. May be one of `rprivate`, `private`, `rshared`, `shared`, `rslave`, `slave`. |

The meaning of each key-value pair depends on the `type` of the mount.
It can be possible to find the meaning of each key-value pair in the following
links:
* [Volumes](https://docs.docker.com/storage/volumes/#choose-the--v-or---mount-flag)
* [Bind mounts](https://docs.docker.com/storage/bind-mounts/#choose-the--v-or---mount-flag)
* [tmpfs mounts](https://docs.docker.com/storage/tmpfs/#choose-the---tmpfs-or---mount-flag)

#### Example

The following example shows how to start a container with a volume.

```bash
$ docker run -dp 127.0.0.1:3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started
```

* `-dp 127.0.0.1:3000:3000` : Publish a container's port `3000` to the host's
    port `3000`.
* `--mount type=volume,src=todo-db,target=/etc/todos` : Mount the volume
    `todo-db` to the container's `/etc/todos` directory.
* `getting-started` : Specify the name of image `getting-started`.

#### Example

The following example shows how to start a container with a bind mount.

```bash
$ docker run -it --mount type=bind,src="$(pwd)",target=/src ubuntu bash
```

* `-it` : Allocate a pseudo-TTY and keep STDIN open even if not attached.
* `--mount type=bind,src="$(pwd)",target=/src` : Mount the current directory
    to the container's `/src` directory.

#### Example

The following example shows how to start a container with a bind mount.

```bash
$ docker run -dp 127.0.0.1:3000:3000 \
    -w /app --mount type=bind,src="$(pwd)",target=/app \
    node:18-alpine \
    sh -c "yarn install && yarn run dev"
```

* `-dp 127.0.0.1:3000:3000` : Publish a container's port `3000` to the host's
    port `3000`.
* `-w /app` : Set the working directory to `/app`.
* `--mount type=bind,src="$(pwd)",target=/app` : Mount the current directory
    to the container's `/app` directory.
* `node:18-alpine` : Specify the name of image `node` with tag `18-alpine`.
* `sh -c "yarn install && yarn run dev"` : Specify a command and argument when
    running the container.

#### Example

```bash
$ docker network create todo-app
$ docker run -d \
    --network todo-app --network-alias mysql \
    -v todo-mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_DATABASE=todos \
    mysql:8.0
```


```bash
$ docker run -dp 127.0.0.1:3000:3000 \
  -w /app -v "$(pwd):/app" \
  --network todo-app \
  -e MYSQL_HOST=mysql \
  -e MYSQL_USER=root \
  -e MYSQL_PASSWORD=secret \
  -e MYSQL_DB=todos \
  node:18-alpine \
  sh -c "yarn install && yarn run dev"
```

### [`docker ps`](https://docs.docker.com/engine/reference/commandline/ps/)

List containers

#### Synopsis

```bash
$ docker ps [OPTIONS]
```

#### `[OPTIONS]`

| Option       | Short | Default | Description                                             |
| ------------ | ----- | ------- | ------------------------------------------------------- |
| `--all`      | `-a`  |         | Show all containers (default shows just running).       |
| `--filter`   | `-f`  |         | Filter output based on conditions provided.             |
| `--format`   |       |         | Pretty-print containers using a custom template.        |
| `--last`     | `-n`  | `-1`    | Show n last created containers (includes all states)    |
| `--latest`   | `-l`  |         | Show the latest created container (includes all states) |
| `--no-trunc` |       |         | Don't truncate output                                   |
| `--quiet`    | `-q`  |         | Only display numeric IDs                                |
| `--size`     | `-s`  |         | Display total file sizes                                |

Available argument for `--format`
: 
* `table`: Print output in table format with column headers (default)
* `table TEMPLATE`: Print output in table format with given [`TEMPLATE`](https://docs.docker.com/config/formatting/)
* `json`: Print in JSON format

#### Example

```bash
$ docker ps
CONTAINER ID   IMAGE             COMMAND                  CREATED         STATUS         PORTS                    NAMES
f24d08c4502d   getting-started   "docker-entrypoint.s…"   3 seconds ago   Up 2 seconds   0.0.0.0:3000->3000/tcp   getting-started-c
```

### Stopping container

[`docker stop`](https://docs.docker.com/engine/reference/commandline/stop/)

```bash
$ docker stop getting-started-c
getting-started-c
```

### [`docker exec`](https://docs.docker.com/engine/reference/commandline/exec/)

Execute a command in running container

#### Synopsis

```bash
$ docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

#### `[OPTIONS]`

| Option          | Short | Description                                          |
| --------------- | ----- | ---------------------------------------------------- |
| `--detach`      | `-d`  | Detached mode: run command in the background         |
| `--detach-keys` |       | Override the key sequence for detaching a container  |
| `--env`         | `-e`  | `API 1.25+` Set environment variables                |
| `--env-file`    |       | `API 1.25+` Read in a file of environment variables  |
| `--interactive` | `-i`  | Keep STDIN open even if not attached                 |
| `--privileged`  |       | Give extended privileges to the command              |
| `--tty`         | `-t`  | Allocate a pseudo-TTY                                |
| `--user`        | `-u`  | Username or UID (format: `<name|uid>[:<group|gid>]`) |
| `--workdir`     | `-w`  | `API 1.35+` Working directory inside the container   |

#### `CONTAINER`

Specify the name or ID of the container.

#### `COMMAND [ARG...]`

Specify a command and argument when running the container.

#### Example

```bash
docker exec <container-id> cat /data.txt
```

* `cat /data.txt` : Specify a command and argument when running the container.

#### Example

The following example shows how to execute a command in a container
which is running a web server. 

```bash
$ docker ps -a                             
CONTAINER ID   IMAGE                   COMMAND                  CREATED       STATUS        PORTS                      NAMES
26ae9db958f4   getting-started         "docker-entrypoint.s…"   2 hours ago   Up 2 hours    127.0.0.1:3000->3000/tcp   dazzling_hamilton
$ docker exec -it dazzling_hamilton sh
/app # ls ../etc/todos -all
total 16
drwxr-xr-x    2 root     root          4096 Jan  6 12:45 .
drwxr-xr-x    1 root     root          4096 Jan  6 10:29 ..
-rw-r--r--    1 root     root          8192 Jan  6 12:45 todo.db
/app # ls ../etc/todos -all
total 16
drwxr-xr-x    2 root     root          4096 Jan  6 12:49 .
drwxr-xr-x    1 root     root          4096 Jan  6 10:29 ..
-rw-r--r--    1 root     root          8192 Jan  6 12:49 todo.db
/app # exit
$
```


### Start stopped container

[`docker start`](https://docs.docker.com/engine/reference/commandline/start/)

```bash
$ docker start getting-started-c
getting-started-c
```

### [`docker rm`](https://docs.docker.com/engine/reference/commandline/rm/)

Remove one or more containers.

#### Synopsis

```bash
$ docker rm [OPTIONS] CONTAINER [CONTAINER...]
```

#### `[OPTIONS]`

| Option      | Short | Default | Description                                             |
| ----------- | ----- | ------- | ------------------------------------------------------- |
| `--force`   | `-f`  |         | Force the removal of a running container (uses SIGKILL) |
| `--link`    | `-l`  |         | Remove the specified link                               |
| `--volumes` | `-v`  |         | Remove the volumes associated with the container        |


#### Example

```bash
$ docker rm getting-started-c
```

* `getting-started-c` : Specify the name of the container. (`CONTAINER`)


## Volume

### [`docker volume create`](https://docs.docker.com/engine/reference/commandline/volume_create/)

Create a volume.

#### Synopsis

```bash
$ docker volume create [OPTIONS] [VOLUME]
```

#### `[OPTIONS]`

| Option                 | Short | Default  | Description                                                                        |
| ---------------------- | ----- | -------- | ---------------------------------------------------------------------------------- |
| `--availability`       |       | `active` | `API 1.42+` `Swarm` Cluster Volume availability (active, pause, drain)             |
| `--driver`             | `-d`  | `local`  | Specify volume driver name                                                         |
| `--group`              |       |          | `API 1.42+` `Swarm` Cluster Volume group (cluster volumes)                         |
| `--label`              |       |          | Set metadata for a volume                                                          |
| `--limit-bytes`        |       |          | `API 1.42+` `Swarm` Minimum size of the Cluster Volume in bytes                    |
| `--name`               |       |          | Specify volume name                                                                |
| `--opt`                | `-o`  |          | Set driver specific options                                                        |
| `--required-bytes`     |       |          | `API 1.42+` `Swarm` Maximum size of the Cluster Volume in bytes                    |
| `--scope`              |       | `single` | `API 1.42+` `Swarm` Cluster Volume access scope (single, multi)                    |
| `--secret`             |       |          | `API 1.42+` `Swarm` Cluster Volume secrets                                         |
| `--sharing`            |       | `none`   | `API 1.42+` `Swarm` Cluster Volume access sharing (none, readonly, onewriter, all) |
| `--topology-preferred` |       |          | `API 1.42+` `Swarm` A topology that the Cluster Volume would be preferred in       |
| `--topology-required`  |       |          | `API 1.42+` `Swarm` A topology that the Cluster Volume must be accessible from     |
| `--type`               |       | `block`  | `API 1.42+` `Swarm` Cluster Volume access type (mount, block)                      |

#### `[VOLUME]`

Specify the name of the volume.

#### Example

```bash
$ docker volume create todo-db
todo-db
```

* `todo-db` : Specify the name of the volume. (`VOLUME`)

### [`docker volume ls`](https://docs.docker.com/engine/reference/commandline/volume_ls/)

List volumes.

#### Synopsis

```bash
$ docker volume ls [OPTIONS]
```

#### `[OPTIONS]`

| Option      | Short | Default | Description                                                                          |
| ----------- | ----- | ------- | ------------------------------------------------------------------------------------ |
| `--cluster` |       |         | API 1.42+ Swarm Display only cluster volumes, and use cluster volume list formatting |
| `--filter`  | `-f`  |         | Provide filter values (e.g. `dangling=true`)                                         |
| `--format`  |       |         | Format output using a custom template:                                               |
| `--quiet`   | `-q`  |         | Only display volume names                                                            |

Available argument for `--format`
: 
* `table`: Print output in table format with column headers (default)
* `table TEMPLATE`: Print output in table format with given [`TEMPLATE`](https://docs.docker.com/config/formatting/)
* `json`: Print in JSON format

### [`docker volume inspect`](https://docs.docker.com/engine/reference/commandline/volume_inspect/)

Display detailed information on one or more volumes.

#### Synopsis

```bash
$ docker volume inspect [OPTIONS] VOLUME [VOLUME...]
```

#### `[OPTIONS]`

| Option     | Short | Default | Description                                   |
| ---------- | ----- | ------- | --------------------------------------------- |
| `--format` | `-f`  |         | Format the output using the given Go template |

Available argument for `--format`
: 
* `table`: Print output in table format with column headers (default)
* `table TEMPLATE`: Print output in table format with given [`TEMPLATE`](https://docs.docker.com/config/formatting/)
* `json`: Print in JSON format

#### `VOLUME`

Specify the name of the volume.

#### Example

```bash
$docker volume inspect todo-db
[
    {
        "CreatedAt": "2024-01-06T10:22:35Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": null,
        "Scope": "local"
    }
]
```

## Network

### [`docker network create`](https://docs.docker.com/engine/reference/commandline/network_create/)

Create a network.

#### Synopsis

```bash
$ docker network create [OPTIONS] NETWORK
```

#### `[OPTIONS]`

| Option          | Short | Default  | Description                                                  |
| --------------- | ----- | -------- | ------------------------------------------------------------ |
| `--attachable`  |       |          | `API 1.25+` Enable manual container attachment               |
| `--aux-address` |       |          | Auxiliary IPv4 or IPv6 addresses used by Network driver      |
| `--config-from` |       |          | `API 1.30+` The network from which to copy the configuration |
| `--config-only` |       |          | `API 1.30+` Create a configuration only network              |
| `--driver`      | `-d`  | `bridge` | Driver to manage the Network                                 |
| `--gateway`     |       |          | IPv4 or IPv6 Gateway for the master subnet                   |
| `--ingress`     |       |          | `API 1.29+` Create swarm routing-mesh network                |
| `--internal`    |       |          | Restrict external access to the network                      |
| `--ip-range`    |       |          | Allocate container ip from a sub-range                       |
| `--ipam-driver` |       |          | IP Address Management Driver                                 |
| `--ipam-opt`    |       |          | Set IPAM driver specific options                             |
| `--ipv6`        |       |          | Enable IPv6 networking                                       |
| `--label`       |       |          | Set metadata on a network                                    |
| `--opt`         | `-o`  |          | Set driver specific options                                  |
| `--scope`       |       |          | `API 1.30+` Control the network's scope                      |
| `--subnet`      |       |          | Subnet in CIDR format that represents a network segment      |

#### `NETWORK`

Specify the name of the network.

#### Example

Create a network named `todo-app`.

```bash
$ docker network create todo-app
```

#### Example

Create a network named `alpine-net` with `bridge` driver.

```bash
$ docker network create --driver bridge alpine-net
```


## Others

### Remove unused data

[`docker system prune`](https://docs.docker.com/engine/reference/commandline/system_prune/)

```bash
$ docker system prune -a
```

## Links

* [Docker CLI cheatsheet](https://docs.docker.com/get-started/docker_cheatsheet.pdf)
