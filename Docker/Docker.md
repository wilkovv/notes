# Docker

### Commands to work with images

```shell
# Lists local images
docker images

# Pulls image from remote repo
docker pull [<repo>:]<image_name>[:<image_tag>]

# Remove local image
docker rmi [<repo>:]<image_name>[:<image_tag>]

# Remove dangling images
docker image prune
```

### Commands to work with containters

```shell
docker run [-d] [-p <host_port>:<container_port>] [-i] [-t] [-v <host_mount_point>:<container_mount_point>]
[-e <ENVIROMENT_VARIABLE>=<value>] [--name <container_name>] [--network <network_name>] 
<image_repo>/<image_name>:<image_tag> [<COMMAND>]
    #[-d] detached mode
    #[-p] binding container port to host port  
    #[-i] interactive mode
    #[-t] pseudoterminal
    #[-v] binding persistent volume from host to container
    #[-e] setting enviroment variables for container
    #[--name] names container with defined value
    #[--network] create container in given network
    #[COMMAND] command to run in container

docker exec [-i] [-t] <container_name> <COMMAND>
    #[-i] interactive mode
    #[-t] pseudoterminal
    #[COMMAND] command to run in container

docker start <container_name>      
docker stop <container_name>

docker attach <container_name>
    # default detach combination is Ctrl+P+Q   
             
docker rm <container_name>                 
docker inspect <container_name>

docker ps [-a] [-q]
    #[-a] lists all containers (includes stopped)
    #[-q] quiet - only display container IDs

docker logs <container_name>

# Removing all containers
docker ps -qa | xargs docker rm
```

### Docker Network Types

- **Default bridge**

- **User-Defined bridge**

- **macvlan** (promiscous mode) - shared network interface with host, *optionally allows creating VLAN 802.1q subinterfaces*

- **ipvlan L2** - shares host MAC address with containers

- **ipvlan L3** - connections are routed through the host interface

- **Overlay Network** - used for networking with multiple docker hosts

- **Null**

### Commands to work with networks

```shell
docker network ls
docker network create <network_name>
docker network rm <network_name>
```

### Docker Volume Types

- Host Volumes

- Anonymous Volumes

- Named Volumes

### Commands to work with volumes

```shell
docker volume ls
docker volume rm <volume_name>
```

# Creating Docker Images

### Dockerfile reference

[Dockerfile reference | Docker Documentation](https://docs.docker.com/engine/reference/builder/#expose)

[Best practices for writing Dockerfiles | Docker Documentation](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

Notes from docs:

- ADD allows to add remote files while COPY does not. 

- CMD might be used to provide arguments to ENTRYPOINT - allows to use container as executable with parameters specified with docker run.

### Example Dockerfile

```dockerfile
# Build image on ubuntu 22.04 as base.
FROM ubuntu:22.04

# Update ubuntu repos and install openjdk.
RUN ["/bin/bash", "-c","apt update && apt install -y openjdk-17-jre-headless"]

# Setup new unprivileged minecraft user with /srv/minecraft directory
RUN ["/bin/bash", "-c","useradd -u 1000 minecraft"]
RUN ["/bin/bash", "-c","mkdir -p /srv/minecraft/data && chown -R minecraft /srv/minecraft"]

# Download official 1.19.2 server jar from mojang servers
ADD --chown=1000 ["https://piston-data.mojang.com/v1/objects/f69c284232d7c7580bd89a5a4931c3581eae1378/server.jar","/srv/minecraft/server.jar"]

# /srv/minecraft/data is mount point for volume with all the server files from host machine
# if no volume is attached all server files will be deleted when container is stopped.
WORKDIR /srv/minecraft/data

# Switch to minecraft user
USER 1000

# By default use 1G of RAM
ENV MEM=1G

# Accept EULA
RUN ["/bin/bash", "-c", "echo 'eula=true' > eula.txt"]

# Start server.jar, redirecting stdin with cat - makes it possible to detach and reattach to 
# minecraft console with 'docker attach', however container must be run with '-it' options.
EXPOSE 25565
CMD ["/bin/bash","-c","cat | java -Xmx$MEM -Xms$MEM -jar /srv/minecraft/server.jar nogui"]
```

### Building Images from Dockerfile:

```shell
# Dockerfile name must be exactly 'Dockerfile' and be present in <build_directory>
docker build -t <image_name>:<image_version> <build_directory>
```

### Working with registry:

```shell
# logging into registry varies by provider
docker login [options] [server]

docker search [options] <term>
docker pull [options] <registry_Domain>/<image_name>[:<image_tag>]

# tagging image for specific repo
docker tag <source_image>[:<image_tag>] <target_image>[:<image_tag>]

docker push [options] <registry_Domain>/<image_name>[:<image_tag>]

```

# Docker compose

### Example yaml config file

```yaml
version: '3'
services:
  
  webserver:
    image: nginx
    volumes:
     - www-data:/usr/share/nginx/html
    networks: 
     - webnetwork
    ports:
     - "8080:80"
  
  minecraft-1:
    image: mcserver:1.9.2
    stdin_open: true
    tty: true
    environment:
     - MEM=2G
    volumes:
     - minecraft-server:/srv/minecraft/data
    networks:
      mcnetwork:
       ipv4_address: "10.10.1.2"

  busybox-1:
    image: busybox
    stdin_open: true
    tty: true
    networks:
      mcnetwork:
       ipv4_address: "10.10.1.3"

networks:
  webnetwork: {}
  mcnetwork: 
    driver: ipvlan
    driver_opts:
      parent: "wlp3s0"
      ipvlan_mode: "l3"
    ipam:
      config:
       - subnet: "10.10.1.0/24"

volumes:
  www-data:
    driver: local
  minecraft-server:
    driver: local
```

### Command to run/stop setup

```shell
docker-compose -f <yaml_file> up|down
```