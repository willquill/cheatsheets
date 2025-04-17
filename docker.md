# Docker Cheatsheet

## Docker Commands

Stop all running containers

`docker stop $(docker ps -aq)`

Kill all running containers

`docker kill $(docker ps -q)`

Delete all stopped containers

`docker rm $(docker ps -a -q)`

Delete all images

`docker rmi $(docker images -q)`

Remove obsolete images

`docker image prune`

Update all images ([source](http://www.googlinux.com/update-all-docker-images/index.html))

`docker images |grep -v REPOSITORY|awk '{print $1}'|xargs -L1 docker pull`

## Docker Compose Commands

Pull latest images

`docker compose pull`

Restart containers

*If you want to stop other containers not in the compose file, add --remove-orphans*

`docker compose up -d`

Update all containers

`docker stop $(docker ps -aq) && docker compose up -d --force-recreate --build`

Update one container

`docker compose up -d --force-recreate --build plex`

Useful aliases for docker compose

```sh
alias dcrun='docker compose up -d'
alias dclogs='docker compose logs -tf --tail="50"'
```

## Create a docker volume of a CIFS share ([source](https://stackoverflow.com/questions/50239386/docker-add-network-drive-as-volume-on-windows))

`docker volume create --driver local --opt type=cifs --opt device=//10.2.200.10/storage --opt o=user=youruser,password=yourpassword storage`

## Space Clearing Methods

```sh
docker system prune --all --volumes --force
```

## More Commands ([source](https://www.reddit.com/r/selfhosted/comments/g3p37k/25_basic_docker_commands_for_beginners/fntnfr9/?context=1))

Creating a Container

`docker create [IMAGE_NAME]`

Creating and Running a Container

`docker run [IMAGE_NAME]`

Starting a Stopped Container

`docker start [CONTAINER_NAME]`

Stopping a Running Container

`docker stop [CONTAINER_NAME]`

Restarting a Running Container

`docker restart [CONTAINER_NAME]`

Pausing a Running Container

`docker pause [CONTAINER_NAME]`

Resuming a Paused Container

`docker unpause [CONTAINER_NAME]`

List currently running containers

`docker ps`

List all containers

`docker ps -a`

Removing a Container

`docker rm [CONTAINER_NAME]`

### Working with Docker Container Images

Docker container images are files that contain the operating system, application and initial state of a docker container. They can be built from Dockerfiles or created from containers that you already have running.

#### Building an Image from a Dockerfile

A dockerfile is a list of commands that docker uses to create and build a container image. You can build an image from a dockerfile by running the below command.

`docker build -f [DOCKERFILE_PATH]`

#### Building an Image from a Container

You can also build an image from a running container. This is a quick to take a backup snapshot of a container that you are working with.

`docker commit [CONTAINER_NAME] [IMAGE_NAME]`

#### Pulling an Image from the Docker Hub

`docker image pull [IMAGE_NAME]`

#### Pushing an Image to the Docker Hub

_first authenticate with:_ `docker login`

`docker image push [IMAGE_NAME]`

#### List Container Images

`docker image ls` or `docker images`

#### Deleting an Image from your System

`docker image remove [IMAGE_NAME]`

### Working with Docker Volumes

Attaching Docker Volumes to containers via the docker run, or docker create commands will allow some of the data in your container to persist across image rebuilds. The following docker commands will help you get started with working with docker volumes.

#### Create a Docker Volume

`docker volume create [VOLUME_NAME]`

#### Remove a Docker Volume

Run the below command to remove a Docker Volume. Remember, if you delete a volume, you will delete any data stored within that volume.

`docker volume rm [VOLUME_NAME]`

#### Inspect a Docker Volume

`docker volume inspect [VOLUME_NAME]`

#### List all Docker Volumes

`docker volume ls`

### Working with Docker Networks

Docker networks determine how containers connect to each other, and the internet. Private networks can be created for various software application stacks to ensure data security.

#### Creating a Docker Network

`docker network create [NETWORK_NAME]`

#### Connecting a Container to a Network

`docker network connect [NETWORK_NAME] [CONTAINER_NAME]`

#### Disconnecting a Container from a Network

`docker network disconnect [NETWORK_NAME] [CONTAINER_NAME]`

#### Inspecting a Network

`docker network inspect [NETWORK_NAME]`

#### Listing all Networks

`docker network ls`

#### Removing a Network

`docker network rm [NETWORK_NAME]`
