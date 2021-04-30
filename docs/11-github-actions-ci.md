# CI with GitHub Actions Workshop

## Preparation

* Make sure to commit your latest `bookinfo-ratings` on Google Cloud Shell to your GitHub Repository
* Make sure your `bookinfo-ratings` branch `master` and `dev` are the same
* Delete all namespaces

```bash
kubectl delete namespace student[X]-bookinfo-dev
kubectl delete namespace student[X]-bookinfo-uat
kubectl delete namespace student[X]-bookinfo-prd
```

### Practice Kubernetes

* Recreate namespace again
  * `student[X]-bookinfo-dev`
  * `student[X]-bookinfo-uat`
  * `student[X]-bookinfo-prd`
* Set default working namespace to `student[X]-bookinfo-dev`
* Create Kubernetes imagePullSecrets
* Create Kubernetes ConfigMap for MongoDB initial database script
* Deploy MongoDB with Helm to be ready for Rating Service

## Create GitHub Workflow

* On Google Cloud Shell
* Prepare master and dev branch to be the same

```bash
cd ~/bookinfo-ratings

# Push latest code on master branch
git checkout master
git pull origin master
git status
git add .
git commit -m "Add latest file"
git push origin master

# Update dev branch
git checkout -b dev
git checkout dev
git pull origin dev
git push origin dev
# Merge master to dev on GitHub
git pull origin dev
```

* If who having problem and want to reset dev branch

```bash
git checkout master
git branch -d dev
git push origin --delete dev
git checkout -b dev
git push origin dev 
```

* Create GitHub Workflows

```bash
mkdir -p .github/workflows/
touch .github/workflows/hello-world.yml
```

* [Note] View > Toggle Hidden Files

```yaml
name: hello-world
on: [push]
jobs:
  print-hello-world:
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - run: echo "Hello World"
      - run: ls -l
```

* Git commit and push
* Check result at <https://github.com/[GITHUB_USER]/bookinfo-ratings/actions>
* Check <https://github.com/marketplace/actions/checkout>

## Build and push Docker Image Action

* Create new workflow file with `touch .github/workflows/dev-env.yml`

```yaml
name: dev-env
on:
  push:
    branches:
      - dev
jobs:
  build:
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - run: docker build -t ghcr.io/[GITHUB_USER]/bookinfo-ratings:dev .
      - run: docker images
      - run: docker push ghcr.io/[GITHUB_USER]/bookinfo-ratings:dev
```

* Check result at <https://github.com/[GITHUB_USER]/bookinfo-ratings/actions>

### Add permission to push Docker Image

* Go to <https://github.com/[GITHUB_USER]?tab=packages> and click on `bookinfo-ratings` package
* Click on `Package Settings` on the right
* Click on menu `Actions access` on the left
* Click on `Add Repository` button
* Search for your `bookinfo-ratings` repository
* Change `Role` to `Write`
* Remove hello-world workflow with `rm .github/workflows/hello-world.yml`
* Update `dev-env.yml`

```yaml
name: dev-env
on:
  push:
    branches:
      - dev
jobs:
  build:
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - run: docker build -t ghcr.io/[GITHUB_USER]/bookinfo-ratings:dev .
      - run: echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
      - run: docker push ghcr.io/[GITHUB_USER]/bookinfo-ratings:dev
```

* Check result at <https://github.com/[GITHUB_USER]/bookinfo-ratings/actions>

### Practice GitHub Actions Marketplace

* Improve step docker login, build, and push to use [Docker Login
](https://github.com/marketplace/actions/docker-login) and [Build and push Docker images](https://github.com/marketplace/actions/build-and-push-docker-images) community actions instead
* Hint

```yaml
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v1
        with:
          ...
      - name: Build and push
        uses: docker/build-push-action@v2
        with:
          ...
```

* You can remove step `action/checkout` since `build-push-action` will checkout git as well

Next: [CD with GitHub Actions Workshop](12-github-actions-cd.md)
