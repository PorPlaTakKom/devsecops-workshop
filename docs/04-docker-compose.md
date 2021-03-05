# Docker Compose Workshop

## Prerequisites

* Linux Terminal, Google Cloud Shell, MacOS Terminal, or WSL2 on Windows
* Docker
* Docker Compose
* Your own text editor such as Vim or VSCode

## Install Docker Compose

```bash
sudo CRYPTOGRAPHY_DONT_BUILD_RUST=1 pip3 install docker-compose
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.28.5/contrib/completion/bash/docker-compose \
  -o /etc/bash_completion.d/docker-compose
docker-compose version
```

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

1. Build and run details services Docker Compose

Next: [Kubernetes Command Line Workshop](05-k8s-cli.md)
