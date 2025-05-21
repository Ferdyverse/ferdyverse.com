---
{"dg-publish":true,"dg-path":"HowTo/docker-tipps","permalink":"/how-to/docker-tipps/","tags":["notes/fern"],"dgShowToc":true,"noteIcon":"fern","created":"2024-05-11 09:52","updated":"2024-07-09 11:08"}
---


> [!warning] Current state
> I just started this page. It will be updated soon.

## Container lifecycle
- [`docker create`](https://docs.docker.com/engine/reference/commandline/create) creates a container but does not start it.
- [`docker rename`](https://docs.docker.com/engine/reference/commandline/rename/) allows the container to be renamed.
- [`docker run`](https://docs.docker.com/engine/reference/commandline/run) creates and starts a container in one operation.
- [`docker rm`](https://docs.docker.com/engine/reference/commandline/rm) deletes a container.
- [`docker update`](https://docs.docker.com/engine/reference/commandline/update/) updates a container's resource limits.

Normally if you run a container without options it will start and stop immediately, if you want keep it running you can use the command, `docker run -td container_id` this will use the option `-t` that will allocate a pseudo-TTY session and `-d` that will detach automatically the container (run container in background and print container ID).

If you want a transient container, `docker run --rm` will remove the container after it stops.

If you want to map a directory on the host to a docker container, `docker run -v $HOSTDIR:$DOCKERDIR`. Also see [[Z/Docker Tipps#Volumes\|Volumes]].

If you want to remove also the volumes associated with the container, the deletion of the container must include the `-v` switch like in `docker rm -v`.

There's also a [logging driver](https://docs.docker.com/engine/admin/logging/overview/) available for individual containers in docker 1.10. To run docker with a custom log driver (i.e., to syslog), use `docker run --log-driver=syslog`.

Another useful option is `docker run --name yourname docker_image` because when you specify the `--name` inside the run command, this will allow you to start and stop a container by calling it with the name you specified when you created it.

### Starting and Stopping
- [`docker start`](https://docs.docker.com/engine/reference/commandline/start) starts a container so it is running.
- [`docker stop`](https://docs.docker.com/engine/reference/commandline/stop) stops a running container.
- [`docker restart`](https://docs.docker.com/engine/reference/commandline/restart) stops and starts a container.
- [`docker pause`](https://docs.docker.com/engine/reference/commandline/pause/) pauses a running container, "freezing" it in place.
- [`docker unpause`](https://docs.docker.com/engine/reference/commandline/unpause/) will unpause a running container.
- [`docker wait`](https://docs.docker.com/engine/reference/commandline/wait) blocks until running container stops.
- [`docker kill`](https://docs.docker.com/engine/reference/commandline/kill) sends a SIGKILL to a running container.
- [`docker attach`](https://docs.docker.com/engine/reference/commandline/attach) will connect to a running container.

If you want to detach from a running container, use `Ctrl + p, Ctrl + q`.

#### Capabilities
Linux capabilities can be set by using `cap-add` and `cap-drop`. See [https://docs.docker.com/engine/reference/run/#/runtime-privilege-and-linux-capabilities](https://docs.docker.com/engine/reference/run/#/runtime-privilege-and-linux-capabilities) for details. This should be used for greater security.

To mount a FUSE based filesystem, you need to combine both --cap-add and --device:
```shell
docker run --rm -it --cap-add SYS_ADMIN --device /dev/fuse sshfs
```

Give access to a single device:
```shell
docker run -it --device=/dev/ttyUSB0 debian bash
```

Give access to all devices:
```shell
docker run -it --privileged -v /dev/bus/usb:/dev/bus/usb debian bash
```

### Info
- [`docker ps`](https://docs.docker.com/engine/reference/commandline/ps) shows running containers.
- [`docker logs`](https://docs.docker.com/engine/reference/commandline/logs) gets logs from container. (You can use a custom log driver, but logs is only available for `json-file` and `journald` in 1.10).
- [`docker inspect`](https://docs.docker.com/engine/reference/commandline/inspect) looks at all the info on a container (including IP address).
- [`docker events`](https://docs.docker.com/engine/reference/commandline/events) gets events from container.
- [`docker port`](https://docs.docker.com/engine/reference/commandline/port) shows public facing port of container.
- [`docker top`](https://docs.docker.com/engine/reference/commandline/top) shows running processes in container.
- [`docker stats`](https://docs.docker.com/engine/reference/commandline/stats) shows containers' resource usage statistics.
- [`docker diff`](https://docs.docker.com/engine/reference/commandline/diff) shows changed files in the container's FS.

`docker ps -a` shows running and stopped containers.

`docker stats --all` shows a list of all containers, default shows just running.

## Images
### Lifecycle
- [`docker images`](https://docs.docker.com/engine/reference/commandline/images) shows all images.
- [`docker import`](https://docs.docker.com/engine/reference/commandline/import) creates an image from a tarball.
- [`docker build`](https://docs.docker.com/engine/reference/commandline/build) creates image from Dockerfile.
- [`docker commit`](https://docs.docker.com/engine/reference/commandline/commit) creates image from a container, pausing it temporarily if it is running.
- [`docker rmi`](https://docs.docker.com/engine/reference/commandline/rmi) removes an image.
- [`docker load`](https://docs.docker.com/engine/reference/commandline/load) loads an image from a tar archive as STDIN, including images and tags (as of 0.7).
- [`docker save`](https://docs.docker.com/engine/reference/commandline/save) saves an image to a tar archive stream to STDOUT with all parent layers, tags & versions (as of 0.7).

### Info
- [`docker history`](https://docs.docker.com/engine/reference/commandline/history) shows history of image.
- [`docker tag`](https://docs.docker.com/engine/reference/commandline/tag) tags an image to a name (local or registry).

### Cleaning up
While you can use the `docker rmi` command to remove specific images,  `docker image prune` is available for removing unused images. See [[Z/Docker Tipps#Prune\|Prune]].

### Load/Save image
Load an image from file:
```shell
docker load < my_image.tar.gz
```

Save an existing image:
```shell
docker save my_image:my_tag | gzip > my_image.tar.gz
```

### Import/Export container
Import a container as an image from file:
```shell
cat my_container.tar.gz | docker import - my_image:my_tag
```

Export an existing container:
```shell
docker export my_container | gzip > my_container.tar.gz
```

#### Difference between loading a saved image and importing an exported container as an image
Loading an image using the `load` command creates a new image including its history.

Importing a container as an image using the `import` command creates a new image excluding the history which results in a smaller image size compared to loading an image.

## Volumes
Docker volumes are [free-floating filesystems](https://docs.docker.com/engine/tutorials/dockervolumes/). They don't have to be connected to a particular container.

### Lifecycle
- [`docker volume create`](https://docs.docker.com/engine/reference/commandline/volume_create/)
- [`docker volume rm`](https://docs.docker.com/engine/reference/commandline/volume_rm/)

### Info
- [`docker volume ls`](https://docs.docker.com/engine/reference/commandline/volume_ls/)
- [`docker volume inspect`](https://docs.docker.com/engine/reference/commandline/volume_inspect/)

Volumes are useful in situations where you can't use links (which are TCP/IP only). For instance, if you need to have two docker instances communicate by leaving stuff on the filesystem.

You can mount them in several docker containers at once, using `docker run --volumes-from`.

Because volumes are isolated filesystems, they are often used to store state from computations between transient containers. That is, you can have a stateless and transient container run from a recipe, blow it away, and then have a second instance of the transient container pick up from where the last one left off.

You can use remote NFS volumes if you're [feeling brave](https://docs.docker.com/engine/tutorials/dockervolumes/#/mount-a-shared-storage-volume-as-a-data-volume).

## Exposing ports
Exposing incoming ports through the host container is [fiddly but doable](https://docs.docker.com/engine/reference/run/#expose-incoming-ports).

This is done by mapping the container port to the host port (only using localhost interface) using `-p`:
```shell
docker run -p 127.0.0.1:$HOSTPORT:$CONTAINERPORT \
  --name CONTAINER \
  -t someimage
```

You can tell Docker that the container listens on the specified network ports at runtime by using [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose):
```dockerfile
EXPOSE <CONTAINERPORT>
```

Note that `EXPOSE` does not expose the port itself - only `-p` will do that.

If you forget what you mapped the port to on the host container, use `docker port` to show it:
```shell
docker port CONTAINER $CONTAINERPORT
```

## Sources
- https://github.com/wsargent/docker-cheat-sheet
- [15 Docker Tips in 5 minutes](http://sssslide.com/speakerdeck.com/bmorearty/15-docker-tips-in-5-minutes)
- [CodeFresh Everyday Hacks Docker](https://codefresh.io/blog/everyday-hacks-docker/)

## Tipps
### Prune
The new [Data Management Commands](https://github.com/docker/docker/pull/26108) have landed as of Docker 1.13:
- `docker system prune`
- `docker volume prune`
- `docker network prune`
- `docker container prune`
- `docker image prune`

### df
`docker system df` presents a summary of the space currently used by different docker objects.

### Last IDs
```shell
alias dl='docker ps -l -q'
docker run ubuntu echo hello world
docker commit $(dl) helloworld
```

### Commit with command (needs Dockerfile)
```shell
docker commit -run='{"Cmd":["postgres", "-too -many -opts"]}' $(dl) postgres
```

### Get IP address
```shell
docker inspect $(dl) | grep -wm1 IPAddress | cut -d '"' -f 4
```

Or with [jq](https://stedolan.github.io/jq/) installed:
```shell
docker inspect $(dl) | jq -r '.[0].NetworkSettings.IPAddress'
```

Or using a [go template](https://docs.docker.com/engine/reference/commandline/inspect):
```shell
docker inspect -f '{{ .NetworkSettings.IPAddress }}' <container_name>
```

Or when building an image from Dockerfile, when you want to pass in a build argument:
```shell
DOCKER_HOST_IP=`ifconfig | grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" | grep -v 127.0.0.1 | awk '{ print $2 }' | cut -f2 -d: | head -n1`
echo DOCKER_HOST_IP = $DOCKER_HOST_IP
docker build \
  --build-arg ARTIFACTORY_ADDRESS=$DOCKER_HOST_IP 
  -t sometag \
  some-directory/
```

### Get port mapping
```shell
docker inspect -f '{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' <containername>
```

### Find containers by regular expression
```shell
for i in $(docker ps -a | grep "REGEXP_PATTERN" | cut -f1 -d" "); do echo $i; done
```

### Get Environment Settings
```shell
docker run --rm ubuntu env
```

### Kill running containers
```shell
docker kill $(docker ps -q)
```

### Delete all containers (force!! running or stopped containers)
```shell
docker rm -f $(docker ps -qa)
```

### Delete old containers
```shell
docker ps -a | grep 'weeks ago' | awk '{print $1}' | xargs docker rm
```

### Delete stopped containers
```shell
docker rm -v $(docker ps -a -q -f status=exited)
```

### Delete containers after stopping
```shell
docker stop $(docker ps -aq) && docker rm -v $(docker ps -aq)
```

### Delete dangling images
```shell
docker rmi $(docker images -q -f dangling=true)
```

### Delete all images
```shell
docker rmi $(docker images -q)
```

### Show image dependencies
```shell
docker images -viz | dot -Tpng -o docker.png
```

### Monitor system resource utilization for running containers
To check the CPU, memory, and network I/O usage of a single container, you can use:
```shell
docker stats <container>
```

For all containers listed by ID:
```shell
docker stats $(docker ps -q)
```

For all containers listed by name:
```shell
docker stats $(docker ps --format '{{.Names}}')
```

For all containers listed by image:
```shell
docker ps -a -f ancestor=ubuntu
```

Remove all untagged images:
```shell
docker rmi $(docker images | grep “^” | awk '{split($0,a," "); print a[3]}')
```

Remove container by a regular expression:
```shell
docker ps -a | grep wildfly | awk '{print $1}' | xargs docker rm -f
```

Remove all exited containers:
```shell
docker rm -f $(docker ps -a | grep Exit | awk '{ print $1 }')
```

### Volumes can be files
Be aware that you can mount files as volumes. For example you can inject a configuration file like this:
```shell
# copy file from container
docker run --rm httpd cat /usr/local/apache2/conf/httpd.conf > httpd.conf

# edit file
vim httpd.conf

# start container with modified configuration
docker run --rm -it -v "$PWD/httpd.conf:/usr/local/apache2/conf/httpd.conf:ro" -p "80:80" httpd
```
## Auto restart container
The second container will restart the first container which is mentioned in the restart command.
```yaml title=docker-compose.yml
services:
  app:
    image: nginx:alpine
    ports: ["80:80"]
    restart: unless-stopped

  restarter:
    image: docker:cli
    volumes: ["/var/run/docker.sock:/var/run/docker.sock"]
    command: ["/bin/sh", "-c", "while true; do sleep 86400; docker restart app_app_1; done"]
    restart: unless-stopped
```
