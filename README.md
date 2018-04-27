# DockerizationWithDocker
Learning path notes.

## Getting Started with Docker

Getting tools: https://docs.docker.com/manuals/

Install Docker (it comes with Docker Compose and Docker Machine)

Output JSON formats: https://github.com/stedolan/jq
https://stedolan.github.io/jq/

### Why Docker?

* Portable
* Low resource
* Scalable
* Isolation
* Self-sufficient
* Immutable: same behavior local, server, CI, etc.

### Operating Containers

```
# This process is short running
$ docker container run --rm mongo

# list running Containers
$ docker container ls -a

# Run detached container (production)
$ docker container run -d --name mongo mongo

# Check the logs of a container
$ docker container logs mongo

# Stop a container
$ docker container stop mongo

# Remove a container from the system
$ docker container rm mongo

# Making the DB accessible
$ docker container run -d --name mongo -p 8081:27017 mongo

# Force remove a container without needing to stop
$ docker container rm -f mongo

# No hard coded public port
$ docker container run -d --name mongo -p 27017 mongo

# Persist data in the local driver
$ docker container run -d --name mongo -p 27017 -v /tmp/mongo:/data/db mongo

# Inspect the container
$ docker container inspect mongo
```

### Using Docker Compose

Instead of using docker commands, define Docker in YAML format and execute YAML file using Docker compose.

```
# help
$ docker-compose --help
$ docker-compose up --help
```

Example:
https://github.com/vfarcic/go-demo

docker-compose.yml
```yaml
version: '2'

services:

  app:
    image: vfarcic/go-demo
    ports:
      - 8080

  db:
    image: mongo:3.2.10
```

```
# Once you have a project with docker-compose.yml file (detached)
$ docker-compose up -d

# check the running containers
$ docker-compose ps

# get the public ports
$ docker container inspect godemo_app_1 | jq '.'

# get the public port for app 1
$ PORT=$(docker container inspect godemo_app_1 | jq -r '.[0].NetworkSettings.Ports."8080/tcp"[0].HostPort')

# Check if server is up
$ curl localhost:$PORT/demo/hello

# Destroy the project (clean up)
$ docker-compose down
```

Documentation:
https://docs.docker.com/compose/compose-file/

## Docker - A Better way to build apps

### Building Images

* `FROM`: Base image used (alpine is a good lightweight linux to use as base)
* `MAINTAINER`: developer
* `RUN`: like bash command
* `EXPOSE`: public ports
* `ENV`: set environment variables
* `CMD`: commands executed when is converted into a container
* `HEALTHCHECK`: execute command every interval
* `COPY`: copy to local

Dockerfile: defines everything the service needs
```
FROM alpine:3.4
MAINTAINER 	Viktor Farcic <viktor@farcic.com>

RUN mkdir /lib64 && ln -s /lib/libc.musl-x86_64.so.1 /lib64/ld-linux-x86-64.so.2

EXPOSE 8080
ENV DB db
ENV SERVICE_NAME go-demo
CMD ["go-demo"]
HEALTHCHECK --interval=10s CMD wget -qO- localhost:8080/demo/hello

COPY go-demo /usr/local/bin/go-demo
RUN chmod +x /usr/local/bin/go-demo
```

Build a binary before
```
$ docker container run -it --rm -v $PWD:/tmp -w /tmp golang:1.6 sh -c "go get -d -v -t && go build -v -o go-demo"
```

Run
```
$ docker image build -t go-demo .
```

List images in the system
```
$ docker image ls
```

https://docs.docker.com/engine/reference/builder/

### Interacting with images (store, pull, run)

* Docker Registry

https://hub.docker.com/

If the image name starts with something that looks like a username, it will push to public registry.
```
$ docker image tag go-demo vfarcic/go-demo

$ docker login

$ docker image push vfarcic/go-demo
```

Storing images (our own registry)
```
$ docker container run -d --name registry -p 5000:5000 registry

$ docker container ls
```

If image name starts with server, it will try to push to that server.
```
$ docker image tag go-demo localhost:5000/go-demo

$ docker image rm localhost:5000/go-demo

$ docker image pull localhost:5000/go-demo

$ docker image tag go-demo localhost:5000/go-demo:1.1

$ docker image push localhost:5000/go-demo:1.1
```
