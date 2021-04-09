# Deploy Rating Service with Helm Chart

## Create Helm Chart for Ratings Service

* Delete all namespaces

```bash
kubectl delete namespace student[X]-bookinfo-dev
kubectl delete namespace student[X]-bookinfo-uat
kubectl delete namespace student[X]-bookinfo-prd
```

### Practice

* Recreate namespace again
  * `student[X]-bookinfo-dev`
  * `student[X]-bookinfo-uat`
  * `student[X]-bookinfo-prd`
* Set default working namespace to `student[X]-bookinfo-dev`
* Create Kubernetes imagePullSecrets
* Create Kubernetes ConfigMap for MongoDB initial database script
* Deploy MongoDB with Helm to be ready for Rating Service

### Create Helm Chart

* `mkdir ~/bookinfo-ratings/k8s/helm` to create directory for Ratings Helm Charts
* Create `Chart.yaml` file inside `helm` directory and put below content

```yaml
apiVersion: v1
description: Bookinfo Ratings Service Helm Chart
name: bookinfo-ratings
version: 1.0.0
appVersion: 1.0.0
home: https://bookinfo.dev.opsta.net/student[X]/ratings
maintainers:
  - name: student[X]
    email: devop[X]@mail.kmutt.ac.th
sources:
  - https://github.com/[USERNAME]/bookinfo-ratings
```

* `mkdir ~/bookinfo-ratings/k8s/helm/templates` to create directory for Helm Templates
* Move our ratings manifest file to template directory with command `mv k8s/dev/ratings-*.yaml k8s/helm/templates/`
* Let's try deploy Ratings Service

```bash
# Deploy Ratings Helm Chart
helm install bookinfo-dev-ratings k8s/helm

# Get Status
kubectl get deployment,pod,service,ingress
```

* Try to access <https://bookinfo.dev.opsta.net/student[X]/ratings/health> and <https://bookinfo.dev.opsta.net/student[X]/ratings/ratings/1> to check the deployment
* Delete previous k8s manifest files

```bash
rm -rf k8s/dev/ k8s/uat/ k8s/prd/
```

## Create Helm Value file for Ratings Service

* Create `values-bookinfo-dev-ratings.yaml` file inside `k8s/helm-values` directory and put below content

```yaml
ratings:
  namespace: student[X]-bookinfo-dev
  image: ghcr.io/[GITHUB_USER]/bookinfo-ratings
  tag: dev
  replicas: 1
  imagePullSecrets: registry-bookinfo
  port: 8080
  healthCheckPath: "/health"
ingress:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  host: bookinfo.dev.opsta.net
  path: "/student[X]/ratings(/|$)(.*)"
  serviceType: ClusterIP
extraEnv:
  SERVICE_VERSION: v2
  MONGO_DB_URL: mongodb://bookinfo-dev-ratings-mongodb:27017/ratings
```

* Let's replace variable one-by-one with these object
  * `{{ .Release.Name }}`
  * `{{ .Values.ratings.* }}`
  * `{{ .Values.ingress.* }}`
* This is sample syntax to have default value

```yaml
{{ .Values.ingress.path | default "/" }}
```

* This is a sample of using if and range syntax

```yaml
        {{- if .Values.extraEnv }}
        env:
        {{- range $key, $value := .Values.extraEnv }}
        - name: {{ $key }}
          value: {{ $value | quote }}
        {{- end }}
        {{- end }}
```

* This is sample syntax to loop annotation

```yaml
  {{- if .Values.ingress.annotations }}
  annotations:
  {{- range $key, $value := .Values.ingress.annotations }}
    {{ $key }}: {{ $value | quote }}
  {{- end }}
  {{- end }}
```

* After replace, you can upgrade release with below command

```bash
helm upgrade -f k8s/helm-values/values-bookinfo-dev-ratings.yaml \
  bookinfo-dev-ratings k8s/helm
```

## Practice

Create Helm value and deploy Rating service for UAT and Production environment

* Deploy MongoDB for each namespace
* Prepare Docker Image for each tag
  * ghcr.io/[GITHUB_USER]/bookinfo-ratings:uat
  * ghcr.io/[GITHUB_USER]/bookinfo-ratings:prd
* Create Helm value file for ratings service for each environment
  * k8s/helm-values/values-bookinfo-dev-ratings.yaml
  * k8s/helm-values/values-bookinfo-uat-ratings.yaml
  * k8s/helm-values/values-bookinfo-prd-ratings.yaml
* Deploy bookinfo on each environment and test it
  * <http://bookinfo.dev.opsta.net/student[X]/ratings/ratings/1>
  * <http://bookinfo.uat.opsta.net/student[X]/ratings/ratings/1>
  * <http://bookinfo.opsta.net/student[X]/ratings/ratings/1>
* Push and tag ratings service repository as `v2.0.0`

## Assignment

* Create Kubernetes & Helm deployment for `details` service
* Deploy details service on 3 environments on each namespaces
  * `student[X]-bookinfo-dev`
  * `student[X]-bookinfo-uat`
  * `student[X]-bookinfo-prd`
* There is no database for details service
* Tag details service repository as `v2.0.0`
