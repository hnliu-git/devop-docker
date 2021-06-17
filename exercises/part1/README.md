## Part1

### 1.1: Getting Started
- Start 3 Containers
- Stop 2 of them leaving 1 up
- Submit the output for `docker ps -a`

Result:
```shell
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                      PORTS     NAMES
72caf513a1cc   nginx          "/docker-entrypoint.…"   47 seconds ago   Exited (0) 14 seconds ago             tender_bhabha
be59e5c4670e   nginx          "/docker-entrypoint.…"   52 seconds ago   Exited (0) 8 seconds ago              modest_driscoll
119a59f626de   nginx          "/docker-entrypoint.…"   58 seconds ago   Up 54 seconds               80/tcp    ecstatic_galileo
```

### 1.2 Cleanup

- Clean the docker daemon from all images and containers 

Result:
```shell
$ docker ps -as
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES     SIZE
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
```

### 1.3 Secret Message
- Submit the secret message and command(s) given as your answer.

Cmds
```shell
$ docker run -d -it --name msg devopsdockeruh/simple-web-service:ubuntu
$ docker exec -it msg bash
root@xxxx: /usr/src/app# tail -f ./text.log
```
Msgs
```shell
Secret message is: 'You can find the source code here: https://github.com/docker-hy'
```

### 1.4 Missing dependencies
- Fix the missing dependency for the function
Cmd
```shell
docker run --rm -it --name browser ubuntu sh -c 'apt-get update; \
                                                 apt-get install --yes curl; \
                                                 echo "Input website:"; \
                                                 read website; \
                                                 echo "Searching.."; \
                                                 sleep 1; \
                                                 curl http://$website;'
```

### 1.5 Sizes of images
- Pull and compare different sizes
```shell
$ docker pull devopsdockeruh/simple-web-service:alpine
REPOSITORY                          TAG       IMAGE ID       CREATED        SIZE
devopsdockeruh/simple-web-service   ubuntu    4e3362e907d5   2 months ago   83MB
devopsdockeruh/simple-web-service   alpine    fd312adc88e0   2 months ago   15.7MB
```
- Test msg func
```shell
docker run -d -it --name img_sz devopsdockeruh/simple-web-service:alpine
docker exec -it img_sz sh
/usr/src/app # cat test.log
```

### 1.6 Hello Docker Hub
- Run `docker run -it devopsdockeruh/pull_exercise`
- Check docker file
    - https://hub.docker.com/r/devopsdockeruh/pull_exercise/dockerfile
```shell
Give me the password: basics
You found the correct password. Secret message is:
"This is the secret message"
```

### 1.7 Two line Dockerfile
- Write a Dockerfile to automatically start the server
Here is the Dockerfile
```shell
FROM devopsdockeruh/simple-web-service:alpine

WORKDIR /usr/src/app

CMD server
```
The console output is:
```shell
$ docker run web-server
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /*path                    --> server.Start.func1 (3 handlers)
[GIN-debug] Listening and serving HTTP on :8080

```

### 1.8 Image for script
- Write a script for a curler
The Dockerfile is:
```shell
FROM ubuntu:18.04

RUN apt-get update \
    &&  apt-get install --yes curl

CMD echo "Input website:" \
    && read website \
    && echo "Searching.." \
    && sleep 1 \
    && curl http://$website
```
The cmd to build and run is:
```shell
$ docker build . -t curler
$ docker run -it curler
```

### 1.9 Volumes
- Start the container with bind mount so that the logs are created into your filesystem.
```shell
docker run -v "$(pwd)/text.log:/usr/src/app/text.log" devopsdockeruh/simple-web-service
```

### 1.10 Ports open
- Map host machine port to a container port
```shell
docker run -p 8080:8080 devopsdockeruh/simple-web-service:alpine serve
```
When I use browser to access `localhost:8080`, the console outputs:
```shell
[GIN-debug] GET    /*path                    --> server.Start.func1 (3 handlers)
[GIN-debug] Listening and serving HTTP on :8080
[GIN] 2021/06/13 - 12:08:51 | 200 |   24.146315ms |      xxx.xx.x.x| GET      "/"
[GIN] 2021/06/13 - 12:08:51 | 200 |      19.578µs |      xxx.xx.x.x| GET      "/favicon.ico"
```

### 1.11 Spring
- Create a Dockerfile for a Java Spring project

Here is the Dockerfile
```shell
FROM openjdk:8

WORKDIR /app

RUN apt-get update && apt-get install git-core --yes

RUN cd /app && git clone https://github.com/docker-hy/spring-example-project.git

RUN cd /app/spring-example-project && ./mvnw package

EXPOSE 8080

ENTRYPOINT ["java", "-jar"]

CMD ["/app/spring-example-project/target/docker-example-1.1.3.jar"]
```

To build and run the image
```shell
docker build . -t spring_project && docker run -p 8080:8080 spring_project
```

### 1.12 Hello, frontend!

- Create a Dockerfile for the project (example-frontend) and give a command so that the project runs in a 
  docker container with port 5000 exposed and published so when you start the container and navigate to 
  http://localhost:5000 you will see message if you’re successful.
  
The docker file is:
```shell
RUN apt-get update && apt-get install git-core curl xsel --yes

# Install node and npm
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash && apt-get update

RUN apt install -y nodejs

RUN node -v && npm -v

# Clone the git project
RUN cd /app && git clone https://github.com/docker-hy/material-applications.git

EXPOSE 5000

# Enter the path
RUN cd /app/material-applications/example-frontend/ \
       && npm install \
       && npm run build \
       && npm install -g serve

ENTRYPOINT ["serve", "-s", "-l", "5000"]

CMD ["/app/material-applications/example-frontend/build/"]
```

To build and run the image
```shell
$ docker build . -t front_end
$ docker run -p 5000:5000 front_end
```

### 1.13 Hello, backend!

- Create a Dockerfile for the project (example-backend) and give a command so that the project runs in 
  a docker container with port 8080 published.

The Dockerfile is:
```shell
FROM golang:1.16

WORKDIR /app

RUN apt-get update && apt-get install git-core --yes

# Clone the git project
RUN cd /app && git clone https://github.com/docker-hy/material-applications.git

EXPOSE 8080

# Enter the path
RUN cd /app/material-applications/example-backend/ \
       && go build \
       && go test ./...

CMD ["/app/material-applications/example-backend/server"]
```

To build, run and test the image
```shell
$ docker build . -t back_end
$ docker run -p 8080:8080 back_end
$ curl http://localhost:8080/ping
pong
```

### 1.14 Environment

- Dockerfile for the front end:

```shell
FROM ubuntu:18.04

WORKDIR /app

RUN apt-get update && apt-get install git-core curl xsel --yes

# Install node and npm
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash && apt-get update

RUN apt install -y nodejs

RUN node -v && npm -v

# Clone the git project
RUN cd /app && git clone https://github.com/docker-hy/material-applications.git

EXPOSE 5000

# Enter the path
RUN cd /app/material-applications/example-frontend/ \
       && npm install \
       && npm run build \
       && npm install -g serve

# Set the backend URI
ENV REACT_APP_BACKEND_URL=http://127.0.0.1:8080

RUN cd /app/material-applications/example-frontend/ && npm run build

ENTRYPOINT ["serve", "-s", "-l", "5000"]

CMD ["/app/material-applications/example-frontend/build/"]
```

- To build and run it
```shell
$ docker build -t front_end:1.14 -f Dockerfile.front .
$ docker run -it -p 5000:5000 front_end:1.14
```

- Dockerfile for the back end:
```shell
FROM golang:1.16

WORKDIR /app

RUN apt-get update && apt-get install git-core --yes

# Clone the git project
RUN cd /app && git clone https://github.com/docker-hy/material-applications.git

EXPOSE 8080

ENV REQUEST_ORIGIN=http://127.0.0.1:5000

# Enter the path
RUN cd /app/material-applications/example-backend/ \
       && go build \
       && go test ./...

CMD ["/app/material-applications/example-backend/server"]
```

- To build and run it
```shell
$ docker build -t back_end:1.14 -f Dockerfile.back .
$ docker run -it -p 8080:8080 back_end:1.14
```


1.15 Homework

Docker Hub link:
- https://hub.docker.com/repository/docker/hnliu/decepticon
