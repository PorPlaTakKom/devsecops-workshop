# Docker Workshop

## Prerequisites

* Linux Terminal, Google Cloud Shell, MacOS Terminal, or WSL2 on Windows
* Docker
* Docker Compose
* Your own text editor such as Vim or VSCode

## Docker Image

```bash
# To show Docker Image on your machine
docker images
# To pull ubuntu image with tag latest from Docker Hub
docker pull ubuntu
# To show your newly pull ubuntu image on your machine
docker images
```

## Docker Image Name and Tag

```bash
# Pull Ubuntu 18.04
docker pull ubuntu:18.04
# To show Docker Image on your machine
docker images
# Pull Ubuntu 20.04
docker pull ubuntu:20.04
# See the Image ID
docker images
```

## Docker Container

```bash
# Run first ubuntu container
docker run ubuntu echo "Hello World"
# Run container with bash command
docker run -i -t ubuntu bash
# Below command are run inside container
whoami
hostname
cat /etc/*release*
exit
```

## Docker Container Basic Operation

```bash
# Show running containers
docker ps
# Show running and stopped containers
docker ps -a
# Run container with specify name
docker run --name ubuntu-universe ubuntu echo "Hello Universe"
docker ps -a
# Delete container by name
docker rm ubuntu-universe
docker ps -a
# Delete container by part of container id
docker rm 07f
docker ps -a
```

## Run Docker as daemon and expose port

```bash
# Run Nginx
docker run nginx:alpine
docker ps -a
# Run Nginx in background
docker run -d nginx:alpine
docker ps
# Export 8080 port from outside forward to port 80 on container
docker run -d -p 8080:80 nginx:alpine
```

* Click on icon `Web preview` and `Preview on port 8080` on the top right of Cloud Shell to access to nginx container

```bash
# What happen if try to expose same port again
docker run -d -p 8080:80 nginx:alpine
# What happen if we expose difference outside port
docker run -d -p 8888:80 nginx:alpine
# You can try Web preview again
docker ps
```

### Docker Exercise

* Try to run Apache Server 2.4.33 on Alpine Linux and expose to port 8083

> Hint: <https://hub.docker.com> and search for apache

## Docker Utilities Commands

```bash
# Rename container name
docker rename vigorous_sammet nginx
# To go inside running container
docker exec -it nginx sh
ls -l
ps -ef
exit
# Show container processes
docker top nginx
# Show logs
docker logs nginx
# Follow logs
docker logs nginx -f
# Try Web preview to see log running
# Show container resource consumes
docker stats
# Show container all metadatas
docker inspect nginx
```

* Delete all the containers

```bash
docker rm -f $(docker ps -aq)
```

## Create Dockerfile for Ratings Service

* Create `Dockerfile` with below content

```Dockerfile
FROM node:14.15.4-alpine3.12

WORKDIR /usr/src/app/

COPY src/ /usr/src/app/
RUN npm install

EXPOSE 8080

CMD ["node", "/usr/src/app/ratings.js", "8080"]
```

* Run below commands to build and run ratings service container

```bash
# Build Docker Image name ratings
docker build -t ratings .
# See newly build Docker Image
docker images
# Run ratings service
docker run -d --name ratings -p 8080:8080 ratings
docker ps -a
# Try Web Preview with /health or /ratings/1 as path
```

* Try to run with SERVICE_VERSION = v2 to force ratings read data from databases

```bash
docker rm -f ratings
docker run -d --name ratings -p 8080:8080 -e SERVICE_VERSION=v2 ratings
docker logs -f ratings
# Try Web Preview with /ratings/1 as path to see error logs
```

## Run MongoDB Container

```bash
# Run MongoDB Container
docker run -d --name mongodb -p 27017:27017 bitnami/mongodb:4.4.4-debian-10-r5
# Test connect
docker exec -it mongodb mongo
show dbs
exit

# Try to run ratings service with MongoDB URL
docker rm -f ratings
docker run -d --name ratings -p 8080:8080 --link mongodb:mongodb \
  -e SERVICE_VERSION=v2 -e 'MONGO_DB_URL=mongodb://mongodb:27017/ratings' ratings
# Try Web Preview with /ratings/1 as path. It still error and causing container exit
```

## Run MongoDB Container with initial database

* Add initial database script

```bash
mkdir ~/ratings/databases
```

* Add [ratings_data.json](../src/ratings/databases/ratings_data.json) and [script.sh](../src/ratings/databases/script.sh) to `databases/` directory
* Run MongoDB Container again

```bash
# Run MongoDB Container
docker rm -f mongodb
docker run -d --name mongodb -p 27017:27017 \
  -v $(pwd)/databases:/docker-entrypoint-initdb.d bitnami/mongodb:4.4.4-debian-10-r5
# Test connect
docker exec -it mongodb mongo
show dbs
use ratings
show collections
db.ratings.find()
exit
```

* Run ratings service again

```bash
docker rm -f ratings
docker run -d --name ratings -p 8080:8080 --link mongodb:mongodb \
  -e SERVICE_VERSION=v2 -e 'MONGO_DB_URL=mongodb://mongodb:27017/ratings' ratings
# Try Web Preview with /ratings/1 as path. It should works now
```

* Update README.md document how to run

````markdown
## How to run with Docker

```bash
docker run -d --name mongodb -p 27017:27017 \
  -v $(pwd)/databases:/docker-entrypoint-initdb.d bitnami/mongodb:4.4.4-debian-10-r5
docker run -d --name ratings -p 8080:8080 --link mongodb:mongodb \
  -e SERVICE_VERSION=v2 -e 'MONGO_DB_URL=mongodb://mongodb:27017/ratings' ratings
```

* Test with path `/ratings/1`
````

* Commit and push code

## Change Docker Command to Docker Compose

* Create `docker-compose.yml` file with below content

```yaml
services:
  ratings:
    build: .
    image: ghcr.io/[GITHUB_USER]/bookinfo-ratings:dev
    ports:
      - "8080:8080"
    environment:
      SERVICE_VERSION: v1
```

* Run Container with Docker Compose command

```bash
# Delete current running container first
docker rm -f ratings mongodb

docker-compose up
```

* Update `README.md` by changing how to run rating service with Docker Compose instead

````markdown
## How to run with Docker

```bash
docker-compose up
```
````

* Push code

### Docker Compose Utility Commands

```bash
# To only build docker image
docker-compose build
# To force build Docker Image everytime
docker-compose up --build
# To choose custom Docker Compose file
docker-compose up -f custom-compose.yml
# To run Docker Compose in background
docker-compose up -d
# To stop all containers after docker-compose up -d
docker-compose stop
# To start all containers after docker-compose stop
docker-compose start
# To restart all containers
docker-compose restart
# To show status of containers
docker-compose ps
# To show all containers processes
docker-compose top
# To list all images in Docker Compose file
docker-compose images
# To stop and clean all docker-compose up resources
docker-compose down
```

### Adding MongoDB to Docker Compose

* Update `docker-compose.yml` file with following content and re-run `docker-compose up` command

```yaml
services:
  ratings:
    build: .
    image: ghcr.io/[GITHUB_USER]/bookinfo-ratings:dev
    ports:
      - "8080:8080"
    environment:
      SERVICE_VERSION: v2
      MONGO_DB_URL: mongodb://mongodb:27017/ratings
  mongodb:
    image: bitnami/mongodb:4.4.4-debian-10-r5
    volumes:
      - "./databases:/docker-entrypoint-initdb.d"
```

### Improve security by using MongoDB Authentication

* Update `databases/script.sh` file with following content

```bash
#!/bin/sh

set -e
mongoimport --host localhost --username $MONGODB_USERNAME --password $MONGODB_PASSWORD \
  --db $MONGODB_DATABASE --collection ratings --drop --file /docker-entrypoint-initdb.d/ratings_data.json
```

* Update `docker-compose.yml` file with following content and re-run `docker-compose up` command

```yaml
services:
  ratings:
    build: .
    image: ghcr.io/[GITHUB_USER]/bookinfo-ratings:dev
    ports:
      - "8080:8080"
    environment:
      SERVICE_VERSION: v2
      MONGO_DB_URL: mongodb://mongodb:27017/ratings
      MONGO_DB_USERNAME: ratings
      MONGO_DB_PASSWORD: CHANGEME
  mongodb:
    image: bitnami/mongodb:4.4.4-debian-10-r5
    volumes:
      - "./databases:/docker-entrypoint-initdb.d"
    environment:
      MONGODB_ROOT_PASSWORD: CHANGEME
      MONGODB_USERNAME: ratings
      MONGODB_PASSWORD: CHANGEME
      MONGODB_DATABASE: ratings
```

* Push code

### Assignment

1. Build and run details services with Dockerfile and Docker Compose
    * Run command `ruby details.rb 8080`
