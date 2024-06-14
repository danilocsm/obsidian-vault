# Basics
---
1. Application for managing builtin containers provided by your operating system
2. More lightweight compared to virtual machines
3. Isolates the application itself, not a whole operating system
4. Its a good tool for setting both development and production enviromnent 

# Images
templates/blueprints for containers
contains code + required tools/runtimes, environment
can be used to run multiple containers with the same dependencies
images are read-only. Any changes to the code, dependencies, the image must be rebuilt

### Dockerfile
File used as an entry to build a new image using Docker.
![[Pasted image 20240529111959.png]]
Layer based structure, each instruction configures a layer that is cached by Docker to optimize when re building the image

builds a new Image based on the Dockerfile present in the directory
(use -t flag to add name and tag)
```bash
docker build -t <name>:<tag> .
```
It is possible to use docker hub (like github) to save your images in repositories. Create an account and repositories to push your local images to it. Pretty straight foward.
# Containers
running "unit of software"
multiple containers can be created based on one image

# Managing data and volumes

There are 3 basic type of datas that a docker container uses:
1. Application data (code + environment) read only stored in the Images
2. Temporary app data (entered user input) read + write stored inside the containers
3. Permanent App data (user accounts) read + write stored with containers & mounted volumes

All data saved inside the container are not lost when the container stops, only when it is removed

Volumes are useful for persisting data even when the containers are removed. The data, although, is not editable
They are folders on the host machine mounted (mapped) inside the container

Volumes are managed by docker

following instruction creates an anonymous volume
```dockerfile
VOLUME ["/folder/inside/container"]
```

list all mounted (anonymous) volumes for running containers
```shell
docker volume ls
```
removes all anonymous volumes
```shell
docker volume prune
```

Creates an anonymous volume
1. Created a specifically for a single container
2. Survives container shutdown / restart unless --rm is used
3. Can not be share across container
4. Since its anonymous, it can't be re-used (even on same image)
5. Useful for preventig data overwritting
6. Can be used to prioritize container-internal paths higher than external paths
```bash
docker run -v /app/data
```
Creates a named volume
1. Create in general - not tied to any specific container
2. Survives container shutdown/restart and removal of the container
3. Can be share across container
4. Can be re-used for same container
```bash
docker run -v data:/app/data
```
Bind mount(NOT MANAGED BY DOCKER)
1. Location on host file system, not tied to any specific container
2. Survives container shutdown/restart and removal of the container
3. Can be share across container
4. Can be re-used for same container
5. Add `:ro` after the container path to make the external path read-only
```bash
docker run -v /path/to/code:/app/code
```

#### Bind Mounts
a folder on the host machine managed by the owner, not by docker
great for persistent and editable data

#### .dockerignore file

Works similiar as .gitignore, but it specifies which folders and files should NOT be copied when building an image with the COPY command

## Docker environment variables

```dockerfile
ENV PORT 80

EXPOSE $PORT
```

```bash
docker run -p 3000:8000 --env PORT=8000 ...
```
setting variables through a file
```bash
docker run -p 3000:8000 --env-file ./.env ...
```

## Docker arguments
```dockerfile
ARG DEFAULT_PORT=80

ENV PORT $DEFAULT_PORT
```

```bash
docker build ... --build-arg DEFAULT_PORT=8000 .
```

## Docker basic CLI commands

list all running containers
```bash
docker ps
```
list all containers
```bash
docker ps -a
```
re starts a non running container (use -a arg to attach)
```bash
docker start <container_id/container_name>
```
stops a running container
```bash
docker stop <container_id/container_name>
```
runs a dettached container exposing a local port to a container port
(use -it flags to make the running container iteractive)
(use --rm flag to automatically remove the container when it stops)
(use --name flag to set a custom name for the container)
```shell
docker run -p local_port:container_port -d <container_id/container_name>
```

(-v flag to create a named volume)
```shell
docker run -p local_port:container_port -v feedback:/app/feedback <container_id/container_name>
```

(-v flag to create a bind mount)
```shell
docker run -p local_port:container_port -v <volume>:<wordir/dir> -v $(pwd):/<container_working_dir> <container_id/container_name>
```

you can override some bind mounts by adding anonymous volumes in the docker run command to preserve some data, like node_modules
```bash
docker run ... -v /app/node_modules 
```
	
attachs to a running dettached container
```bash
docker attach <container_id/container_name>
```
checks logs from the container
```bash
docker logs -f <container_id/container_name>
```
removes the container
```bash
docker rm <container_name/container_id> <container_name/container_id>
```
remove images
```bash
docker rmi <container_name/container_id> <container_name/container_id>
```
list all available images
```bash
docker images
```
removes all non used images
```bash
docker images prune
```
lists a bunch of metadata about the image (very helpful to debug)
```bash
docker image inspect <image_id>
```
copying files between local machine and containers
```bash
docker cp <folder_local> <container_name>:<folder_inside_container>
docker cp <container_name>:<folder_inside_container> <local_folder>
```
pull and push images from/to the default image registry (Docker hub)
```bash
docker push <host_name>:<tag_name>
docker pull <host_name>:<tag_name>
```
rename a tag
```shell
docker tag <old_name>:<old_tag> <new_name>
```
