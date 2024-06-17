---
{"dg-publish":true,"dg-path":"HowTo/docker-tipps","permalink":"/how-to/docker-tipps/","tags":["📝/🌿"],"noteIcon":"fern","created":"2024-05-11T09:52","updated":"2024-06-17T23:46"}
---


> [!warning] Current state
> I just started this page. It will be updated soon.
## Basic commands
```shell
# Show running containers
docker ps

# Pull docker compose container
docker compose pull

# Start docker conatiners with compose and detach them into the background
docker compose up -d

# Clean old images
docker image prune

# Export Image to file
docker save -o <filename>.tar <imagename>

# Import Image from file
docker load -i <filename>.tar
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