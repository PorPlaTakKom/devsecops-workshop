# Docker Workshop

## Prerequisites

* Linux Terminal, Google Cloud Shell, MacOS Terminal, or WSL2 on Windows
* Docker
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

* Add [ratings_data.json](https://github.com/opsta/bookinfo/blob/main/src/ratings/databases/ratings_data.json) and [script.sh](https://github.com/opsta/bookinfo/blob/main/src/ratings/databases/script.sh) to `databases/` directory
* Run MongoDB Container again

```bash
cd ~/ratings/
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
# Build Docker Image for rating service
docker build -t ratings .

# Run MongoDB with initial data in database
docker run -d --name mongodb -p 27017:27017 \
  -v $(pwd)/databases:/docker-entrypoint-initdb.d bitnami/mongodb:4.4.4-debian-10-r5

# Run ratings service on port 8080
docker run -d --name ratings -p 8080:8080 --link mongodb:mongodb \
  -e SERVICE_VERSION=v2 -e 'MONGO_DB_URL=mongodb://mongodb:27017/ratings' ratings
```

* Test with path `/ratings/1` and `/health`
````

* Commit and push code

## Assignment

1. Build and run details services with Dockerfile
   * Source code here <https://github.com/opsta/bookinfo/tree/main/src/details>
   * Run command `ruby details.rb 8080`

Next: [Docker Compose Workshop](04-docker-compose.md)
