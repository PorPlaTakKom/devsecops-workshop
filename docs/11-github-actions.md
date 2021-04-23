# CI/CD with GitHub Actions Workshop

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
  deploy-dev:
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
  deploy-dev:
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

## Deploy Ratings Service to GCP

### Create GKE Credential Secret

* Go to <https://github.com/[GITHUB_USER]/bookinfo-ratings/settings/secrets/actions>
* Click on `New repository secret`
  * `Name:` GKE_CREDENTIALS
  * `Value:` [ASK_FROM_INSTRUCTOR]

### Improve dev-env.yml

```yaml
...
      - name: get-credentials
        uses: google-github-actions/get-gke-credentials@main
        with:
          cluster_name: k8s
          location: asia-southeast1-a
          credentials: ${{ secrets.GKE_CREDENTIALS }}
      - name: deploy
        uses: 'deliverybot/helm@v1'
        with:
          helm: helm3
          release: bookinfo-dev-ratings
          namespace: student[X]-bookinfo-dev
          chart: k8s/helm
          value-files: k8s/helm-values/values-bookinfo-dev-ratings.yaml
```

* Check result at <http://bookinfo.dev.opsta.net/student[X]/ratings/health> and <http://bookinfo.dev.opsta.net/student[X]/ratings/ratings/1> to check the deployment

## Test and Fix CI/CD

* Try to change source code `src/ratings.js` line 207 to `Ratings is working` instead
* Push code, check GitHub Workflow and test health check page result
* Health check page won't update because when helm upgrade new release, everything still the same.

### Fix health check page won't update

* Edit `k8s/helm-values/values-bookinfo-dev-ratings.yaml` and put `COMMIT_SHA: CHANGEME` in `extraEnv:`
* Add following to deploy steps in `dev-env.yml`

```yaml
          values: |
            COMMIT_SHA: $GITHUB_SHA
```

* Push code, check GitHub Workflow and test health check page result again.

## Practice GitHub Actions

* Add [Endpoint check](https://github.com/marketplace/actions/endpoint-check) to do acceptance test on the last step to see health check page is working
* Create `uat-env.yml` workflow but event trigger from master branch (Don't forget to prepare ConfigMap, Secret, and deploy MongoDB in the namespace first)
* Test by pull request and merge from dev to master branch

## Assignment

* Try to use only 1 workflow file with 1 job to follow DRY principle
* Hints

```yaml
name: deploy-non-prd
on:
  push:
    branches:
      - dev
      - master
jobs:
  deploy-non-prd:
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Set env
        run: echo "ENV_NAME=$( [ "$GITHUB_REF" == "refs/heads/master" ] && echo 'uat' || echo ${GITHUB_REF##*/} )" >> $GITHUB_ENV
      ...
```

  * Then use `${{ env.ENV_NAME }}` to refer to environment name
