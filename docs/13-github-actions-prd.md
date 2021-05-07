# Production Deployment with GitHub Actions Workshop

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
* Deploy following on dev, uat, and prd environments
  * Create Kubernetes imagePullSecrets
  * Create Kubernetes ConfigMap for MongoDB initial database script
  * Deploy MongoDB with Helm to be ready for Rating Service
  * Deploy Ratings Service on dev and uat environments with GitHub Action

## Create Tagging Workflows

* Create new workflow file with `touch .github/workflows/tagging.yml`

```yaml
name: tagging
on: 
  workflow_dispatch:
    inputs:
      tag-version:
        description: 'Version to tag'
        required: false
jobs:
  git-tag:
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Git tagging
        id: git-tag
        uses: anothrNick/github-tag-action@1.35.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CUSTOM_TAG: ${{ github.event.inputs.tag-version }}
          DEFAULT_BUMP: patch
    outputs:
      tag-version: ${{ steps.git-tag.outputs.new_tag }}
  docker-tag:
    needs: git-tag
    runs-on: ubuntu-20.04
    steps:
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v1
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Pull and push tagged Docker Image
        uses: akhilerm/tag-push-action@v1.0.0
        with:
          src: ghcr.io/[GITHUB_USER]/bookinfo-ratings:uat
          dst: |
            ghcr.io/[GITHUB_USER]/bookinfo-ratings:${{needs.git-tag.outputs.tag-version}}
```

* Push to `dev` branch and doing pull request to `master` branch
* To manual run tagging workflow
  * Go to <https://github.com/[GITHUB_USER]/bookinfo-ratings/actions/workflows/tagging.yml> or go to your repository > `Actions` > `Workflows tagging` and click on `Run workflow` button
  * You can click on `Run workflow` to make GitHub Actions run tagging workflows
* After workflow finished, go check your release at <https://github.com/[GITHUB_USER]/bookinfo-ratings/releases> to see auto-tagging
* Go check your Docker Image at <https://github.com/users/[GITHUB_USER]/packages/container/package/bookinfo-ratings> to see new tag

## Create Ratings Service Production Environment Workflows

* Create new workflow file with `touch .github/workflows/deploy-prd.yml`

```yaml
name: deploy-prd
on:
  workflow_dispatch:
    inputs:
      deploy-version:
        description: 'Version to deploy'
        required: true
jobs:
  deploy-prd:
    runs-on: ubuntu-20.04
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        with:
          ref: refs/tags/${{ github.event.inputs.deploy-version }}
      - name: get-credentials
        uses: google-github-actions/get-gke-credentials@main
        with:
          cluster_name: k8s
          location: asia-southeast1-a
          credentials: ${{ secrets.GKE_CREDENTIALS }}
      - name: deploy
        uses: deliverybot/helm@v1
        with:
          helm: helm3
          release: bookinfo-prd-ratings
          namespace: student[X]-bookinfo-prd
          chart: k8s/helm
          value-files: k8s/helm-values/values-bookinfo-prd-ratings.yaml
          values: |
            {
              "ratings": {
                "tag": "${{ github.event.inputs.deploy-version }}"
              },
              "extraEnv": {
                "COMMIT_SHA": "${{ github.sha }}"
              }
            }
```

* Push to `dev` branch and doing pull request to `master` branch
* To manual run deploy to production workflow
  * Go to <https://github.com/[GITHUB_USER]/bookinfo-ratings/actions/workflows/deploy-prd.yml> or go to your repository > `Actions` > `Workflows deploy-prd` and click on `Run workflow` button
  * Put version number that you tagged before and click on `Run workflow` to make GitHub Actions run deploy to production workflow
* Check result at <http://bookinfo.opsta.net/student[X]/ratings/health> and <http://bookinfo.opsta.net/student[X]/ratings/ratings/1>

## Practice GitHub Actions

### Add Acceptance Test

* Add another job to do following curl command for acceptance test to see health check page is working

```bash
curl http://bookinfo.dev.opsta.net/student[X]/ratings/health | grep "Ratings is working"
```

## Testing CI/CD from the start

* Try to change source code `src/ratings.js` line 207 to `Ratings is working!!!`
* Push code, check GitHub Workflow and test health check page result on dev environment
* Doing pull request from `dev` branch to `master` branch and check the workflow that will deploy to UAT environment
* Manually trigger tagging workflow and specify version `4.0.0` and see the result
* Manually deploy to production environment by specify version `4.0.0` and check the result

## Assignment

* Create GitHub Actions for `details` service
* Deploy details service with GitHub Workflows on 3 environments on each namespaces
  * `student[X]-bookinfo-dev`
  * `student[X]-bookinfo-uat`
  * `student[X]-bookinfo-prd`
* There is no database for details service
* Tag details service repository as `4.0.0`
