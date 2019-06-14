## Docker Fundamentals
software container platform. 

VM vs. Container. in VM, Application runs with libraries installed in VM on VM OS, which is on top of Hypervisor. A container runs with its libraries in its own image within docker daemon process, which is run on the Host OS.

Docker container often runs on one linux distro, and that is alpine

add user with docker privilege:
```
sudo usermod -aG doker $USER
```

to install a image from image repo:
```
docker run hello-world
```

#### Docker CLI
container has its own isolated file system, networking, process tree separated from the host

the docker run command:
```
docker run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

example:
```
#run in detached mode -d with image to be named nginx (optional) that map <host port>:<container port> of 80:80 using image nginx with tag alpine
docker run -d --name=nginx -p 80:80 nginx:alphine
```

docker is refactoring CLI. the new way to specify docker run is:
```
docker container run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
```

list all containers that are running:
```
docker container ls
```

another example:
```
# added -v to specify mounting a volume, $(pwd) is current directory, mounting to /usr/share/nginx/html inside the container
docker container run --name website -v $(pwd):/usr/share/nginx/html -d -p 83:80 nginx:alpine 
```

the -d is important, if not running as daemon, docker is going to run on the terminal the command is executed

to list all containers
```
# will also show offline containers
docker container ls -a
```

example:
```
# added --rm, so container is automatically removed and will not show up as dead when doing docker container -a
docker run --name ping --rm alpine ping -c 10 www.google.com
```

the old way of listing containers:
```
docker ps -a
```

getting a container shell:
```
# -it means direct tty meaning terminal is connected into the container, and the command is sh meaning you're getting a shell, alpine is the linux OS
docker run --name inside-container --rm -it alpine sh
```

in alpine OS, apk is the apt-get command

docker top displays the resource usage of the container
```
docker top containername
```

to see resource utilization
```
docker stats [OPTIONS] [CONTAINER...]
```

to see logs
```
docker logs [OPTIONS] CONTAINER
```

to get continuous logs since 1 minute ago, showing timestamps
```
docker logs -f -t --since 1m nginx
```

run a new command in an existing container
```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

use docker exec to attach to a container:
```
# execute bash in the container alpine and attach tty with current terminal
docker exec -it alpine bash
```

docker volume command deals with the volume of the container:
```
docker volume COMMAND
```
commands can be:
* create: to create a volume
* inspect: to display detailed information on one or more volumes
* ls: list volumes
* prune: remove all unused volumes
* rm: remove one or more volumes

example create volume:
```
# local means driver is a local file system directory, the name will be my-volume
docker volume create -d local my-volume
```

to see all volumes created:
```
docker volume ls
```

to inspect details of a volume:
```
docker volume inspect my-volume
```

to mount the volume into a container:
```
# mount my-volume under /webapp for this container web running training/webapp with command python app.py
docker run -d -v my-volume:/webapp --name web training/webapp python app.py
```

when container is removed, the volume still exist.

to remove all volumes not mounted by any containers:
```
docker volume prune
```

docker can mount volume in many different places, such as digital ocean, S3 etc

docker update command updates the container configuration.
```
docker update [OPTIONS] CONTAINER [CONTAINER...]
```

example to make a container always restart:
```
# update the container routing to always restart when stopped
docker update --restart=always routing
```

example update memory limit
```
docker update --kernel-memory 80M routing
```

docker start command restart a stopped container
```
docker start CONTAINER
```

docker system command manages docker
```
docker system COMMAND
```
where commands can be:
* df: show docker disk usage
* events: get real time events from server
* info: display system-wide information
* prune: remove unused data

docker network command help isolate container to private network, or isolate databbase connection to private network, and provide DNS names inside the same private network
```
docker network COMMAND
```
where command can be:
* connect: connect a container to network
* create: create a network
* disconnect: disconnect a container from a network
* inspect: display detailed information on one or more networks
* ls: list networks
* prune: remove all unused networks
* rm: remove one or more networks

docker comes with 3 default networks:
* bridge, the default network, when running a container it runs on bridge network
* host, the host's network, the container and the machine will share the same network
* none, not connect to outside network, can disconnect container from external network

to connect a container to a custom network example:
```
#create container nginx-bridge and connect it to a custom network mybridge instead of the default bridge network
docker run -d --name nginx-bridge --network mybridge nginx:alpine
```

#### docker compose

docker compose is a tool for defining and running multi-container docker applications. with compose, you use a compose file to configure the application's services, then a single command to create and start all the services based on the configuration.

docker compose is not part of docker, but a standalone github repo.

example docker compose yaml:
```
version: '3'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
```

to start services in yaml using docker-compose:
```
# will use docker-compose.yaml by default
docker-compose up -d 
```

#### Docker Registry
the registry is a stateless, highly scalable server side application that stores docker images

docker hub provides open registry, free if image is open to the world.

[docker hub](hub.docker.com)
[docker store](store.docker.com) help search for docker images

#### Building Docker Image
docker can build images automatically by reading the instructions fro ma dockerfile

dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image.

to build a image:
```
#build and send to registry (custom) registry.docker.com for alpine linux
docker build -t registry.docker.com/alpine
```

example dockerfile to build a default image
```
FROM alpine:latest
```

example of building a custom docker image: [link](https://github.com/albertogviana/docker-routing-mesh)

```
FROM golang:alpine AS build

ENV APP_VERSION 1.0.0

WORKDIR /go/src/docker-routing-mesh
COPY . .

RUN go build -o docker-routing-mesh

EXPOSE 8080

HEALTHCHECK --interval=10s CMD wget -q0- localhost:8080/heahlth

CMD /go/src/docker-routing-mesh/docker-routing-mesh
```

example command to build the image:
```
docker build -t docker-routing-mesh:latest -f Dockerfile.go
```

docker images command to see all images available
```
docker images
```

#### Building a Docker Image
to build a hello world docker image, create a hello executable. if hello is build in C/C++, it need to specify compile parameters `-static` and `-nostartfiles`

example docker file:
```
FROM alpine:latest
ADD hello /
CMD ["/hello"]
```

use this Dockerfile to build the image using the following command:
```
docker build --tag hello .
```

once the image is built, you can find the image in docker using command
```
docker images
```

then the image can be run using
```
docker run --rm hello
```

notice the image runs and execute the hello executable, print the output to stdout, then it quits

#### Docker Image Build Reference
docker build command takes a dockerfile and a context; the context is either a local filesystem directory containing the image content, or a URL repository

`docker build -f <PATH>` builds the docker image from anywhere in filesystem PATH, `docker build -t <URL>` builds image from remote repository, `-t` can be specified multiple times to take multiple repositories.

to speed up docker builds, it can build from cache. to use cache, specify `--cache-from`

general format of Dockerfile is 
```
# COMMENT
INSTRUCTION arguments
```

the first set of instructions specify the base image the docker image is going to build from. This means `FROM` instruction is used at the begining of the Dockerfile.

##### Dockerfile Parser directives
to deal with different operating system, such as windows, docker has parser directives to deal with special characters used in the Dockerfile
directives appear in the form of

```
# directive_name=value
```

where directive_name is built-in. such as `escape` to define the escape character used in the docker file

##### Dockerfile Environment Replacement
Dockerfile supports picking up variables from shell environment using the same syntax, such as `${variable}`, also supports `${variable:-default}` which is the bash syntax in case variable is not defined, default value is used.

##### ENV Instruction
defines environment variable within the Dockerfile. e.g.

```
ENV foo /bar #defines variable foo to have value /bar
```

environment varible is supported in the following list of instructions:
```
ADD
COPY
ENV
EXPOSE
FROM
LABEL
STOPSIGNAL
USER
VOLUME
WORKDIR
```

notice, `CMD` and `RUN` does not support environment variable.

##### Dockerfile .dockerignore file
defines a list of files in the local filesystem to be ignored when building the image. pattern matching can be used in files/path to be ignored. e.g.

```
# comment
*.md
!README.md
```

ignores all *.md except for README.md file in repository. `!` means exception in the pattern

##### Instruction FROM
specify the base image. `FROM` can be specified multiple times, each override the previous FROM command parameters. The only other instruction that can go before `FROM` is `ARG` which sets some specific property of the base image specified in `FROM`

```
FROM <image> [AS <name>]
FROM <image>[:<tag>] [AS <name>]
FROM <image>[@<digest>] [AS <name>]
```

name alias of the base image may be used in following instructions

##### Instruction ARG
specify additional variables to be declared before the FROM instruction. `ARG` must go before `FROM` instruction

##### Instruction RUN
`RUN` instruction executes command during image build process on top of the base image specified by `FROM`. Result from `RUN` instruction is saved into the image. This means `RUN` instruction may be cached and does not need to be executed if cache is used for image build.

The default shell used for running `RUN` instruction can be changed using `SHELL` instruction.

```
RUN <command>
RUN ["shell_executable", "param1", "param2" ...]
```

run can also choose specific shell in the `[]` form where the first string is a shell executable. notice `[]` must quote each parameter using double quote, not single quote

to force `RUN` command not to use cache, use `docker build --no-cache` option

##### Instruction CMD
`CMD` executes the command on the built image, not during image build.

```
CMD ["shell_executable", "param1", "param2", ... ]
CMD ["param1", "param2", ... ]
CMD <command> <param1> <param2> ...
```

the main purpose of `CMD` is to set defaults for an executing container. the array form is the preferred format for `CMD`. If the container should run the same executable every time, then `CMD` should combine with `ENTRYPOINT` instruction

##### Instruction LABEL
add metadata to the built image

```
LABEL <key>=<value> <key>=<value> ...
```

labels are shown when `docker inspect <image>` command is used

##### Instruction EXPOSE
`EXPOSE` instruction informs the container listens on specified network ports at runtime. TCP is the default protocol if protocol is not specified. `EXPOSE` does not publish the port. it only function as documentation indicating which port is intended to be published. to publish the port of the container, use the -p parameter.

```
EXPOSE <port> [<port>/<protocol> ...]
```

##### Instruction ENV
assign value to environment varaible key during image build.

```
ENV <key> <value>
ENV <key>=<value>
```

##### Instruction ADD
`ADD` instruction copies files/directories or remote URL from `<src>` to filesystem of the image at `<dest>`. `<src>` may contain wild cards. `<dest>` must be either absolute path or relative path to WORKDIR.

```
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] [<src>, ... <dest>] #this form is required for PATH containing white space
```

all copied files will have UID/GID of value 0. unless chown is used. to use chown will require `/etc/passwd` and `/etc/groups` to exist on the image, otherwise `ADD` will fail.

ADD will reuse from cache, even if the URL source location is updated.

##### Instruction COPY
`COPY`instruction copies file/directories from `<src>` and add them to the filesystem of the container directory `<dest>`

```
COPY [--chown=<user>:<group>] <src> ... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"] # this form is required if path contains white space
```

##### Instruction ENTRYPOINT
`ENTRYPOINT` will configure a container to run as an executable. for example, when running the nginx image `docker run -i -t --rm -p 80:80 nginx`, command line arguments after run will be appended after all `ENTRYPOINT` instructions on the image, and will override all elements specified using `CMD`. This allows argument to be passed to the entry point, i.e. `docker run <image> -d ` will pass the `-d` argument to the entry point. You can override the `ENTRYPOINT` instruction using `docker run --entrypoint`

```
ENTRYPOINT ["executable", "param1", "param2" ... ] # prefferred form
ENTRYPOINT command param1 param2 ...               # shell form
```

the shell form prevents any `CMD` and `RUN` command line arguments from being used, but also executes the entrypoint using `bin/sh`, which does not pass signal, which means when doing `docker stop <container>`, the SIGTERM sent to the command is lost.

only the last `ENTRYPOINT` instruction in the docker file has an effect.

`ENTRYPOINT` interact with `CMD` instruction.

* `ENTRYPOINT` should be defined when using container as an executable. 
* `CMD` should be used as way to define default arguments for `ENTRYPOINT` command or executing an ad-hoc command on the container.
* `CMD` will be overridden when running the container with alternative arguments.

`ENTRYPOINT` in shell form overrides all `CMD` instructions, and does not let them get executed

##### Instruction VOLUME
volume instruction creates a mount point with a specified name and marks it as holding externally mounted volume from native host or other container. 

```
VOLUME ["<dir>"]
```

##### Instruction USER
`USER` command sets the username and group to use when running the image and for any `RUN`, `CMD`, and `ENTRYPOINT` instruction that follows it in the Dockerfile.

```
USER <user>[:<group>]
USER <UID>[:<GID>]
```

##### Instruction WORKDIR
`WORKDIR` instruction sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT` that follows it. `WORKDIR` can be specified multiple times, if a relative path is used, it will be relative to the previous `WORKDIR`

```
WORKDIR /path/to/workdir
```

##### Instruction ARG
`ARG` defines variable used at build time to the builder. docker build command can specify additional `ARG` by doing `--build-arg <varname>=<value>`
 
```
ARG <name>[=<default value>]
```

variables defined using `ENV` always override `ARG` of the same name.

list of predefined `ARG`:
* HTTP_PROXY
* http_proxy
* HTTPS_PROXY
* https_proxy
* FTP_PROXY
* ftp_proxy
* NO_PROXY
* no_proxy

`ARG` is not persisted into the build as `ENV` are.

##### Instruction STOPSIGNAL
sets the system call signal to be sent to the conatiner when we exit.

```
STOPSIGNAL signal #where signal can be SIGKILL
```

##### Instruction HEALTHCHECK
`HEALTHCHECK` tells docker how to check for container health when it is running. It can be used to detect problems such as web server stuck in infinite loop, but the process is still running. When healthcheck fails on a container, it is labeld as unhealthy, otherwise it is labeled as healthy after start.

```
HEALTHCHECK [OPTIONS] CMD command
HEALTHCHECK NONE
```

healthcheck options are:
* `--interval=DURATION` default 30 seconds
* `--timeout=DURATION` default 30 seconds
* `--start-period=DURATION` default 30 seconds
* `--retries=N` default 3

there can only be 1 `HEALTHCHECK` instruction in a docker image. the command used for healthcheck should return
* 0 for success
* 1 for unhealthy
* 2 is reserved, do not use this exit code

##### Instruction SHELL
`SHELL` changes the default shell used for shell forms of any command

```
SHELL ["executable", "parameters"]
```

##### Build secrets into docker container
1. When you run a container, it gets its own network namespace in the kernel, with its own network interfaces and corresponding IP addresses.
2. Containers can choose to join the network of an existing container.
3. docker build has a --network argument that lets RUN build steps join a particular network—including that of an existing container.

example:
```
$ cat secret.txt
gadzooks123
$ docker run --name=secrets-server --rm --volume $PWD:/files \
      busybox httpd -f -p 8000 -h /files
```

```
FROM busybox
RUN echo "The secret is: " && \
    wget -O - -q http://localhost:8000/secret.txt
```

```
$ docker build --network=container:secrets-server .
```

### Reference
[Dockerfile Documentation](https://docs.docker.com/engine/reference/builder/#usage)
