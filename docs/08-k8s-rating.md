# Deploy Bookinfo Rating Service on Kubernetes Workshop

## Prepare Kubernetes Environment

```bash
gcloud container clusters get-credentials k8s --project zcloud-cicd --zone asia-southeast1-a
kubectl create namespace student[X]-bookinfo-dev
kubectl config set-context $(kubectl config current-context) --namespace=student[X]-bookinfo-dev
```

## Install Docker Compose

```bash
sudo CRYPTOGRAPHY_DONT_BUILD_RUST=1 pip3 install docker-compose
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.28.5/contrib/completion/bash/docker-compose \
  -o /etc/bash_completion.d/docker-compose
docker-compose version
```

## Prepare Rating Repository

```bash
git clone git@github.com:[GITHUB_USER]/bookinfo-ratings.git
cd bookinfo-ratings
```

* Make sure you have code structure like this repository <https://github.com/opsta/bookinfo/tree/main/src/ratings>
* Make sure `Dockerfile` has content as following

```Dockerfile
FROM node:14.15.4-alpine3.12

WORKDIR /usr/src/app/

COPY src/ /usr/src/app/
RUN npm install

EXPOSE 8080

CMD ["node", "/usr/src/app/ratings.js", "8080"]
```

* Make sure `docker-compose.yml` as content as following

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

* Build Docker Image for Rating service and push to GitHub

```bash
docker-compose build

# You already logged-in to GitHub Registry from last workshop
docker push ghcr.io/[GITHUB_USER]/bookinfo-ratings:dev
```

## Create Secret to pull Docker Image from GitHub Docker Private Registry

```bash
# See the Docker credentials file
cat ~/.docker/config.json

# Show secret
kubectl get secret

# Create Docker credentials Kubernetes Secret
kubectl create secret generic registry-bookinfo \
  --from-file=.dockerconfigjson=$HOME/.docker/config.json \
  --type=kubernetes.io/dockerconfigjson

# See newly created secret
kubectl get secret
kubectl describe secret registry-bookinfo
```

## Create Rating Service Kubernetes Manifest File

* `mkdir -p k8s/` to make a directory to store manifest file
* Create `ratings-deployment.yaml` file inside `k8s/` directory with below content

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bookinfo-dev-ratings
  namespace: student[X]-bookinfo-dev
  labels:
    app: bookinfo-dev-ratings
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bookinfo-dev-ratings
  template:
    metadata:
      labels:
        app: bookinfo-dev-ratings
    spec:
      containers:
      - name: bookinfo-dev-ratings
        image: ghcr.io/[GITHUB_USER]/bookinfo-ratings:dev
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: web-port
          protocol: TCP
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
            scheme: HTTP
        env:
        - name: SERVICE_VERSION
          value: v1
      imagePullSecrets:
      - name: registry-bookinfo
```

* Create `ratings-service.yaml` file inside `k8s/` directory with below content

```yaml
apiVersion: v1
kind: Service
metadata:
  name: bookinfo-dev-ratings
  namespace: student[X]-bookinfo-dev
spec:
  type: ClusterIP
  ports:
  - port: 8080
  selector:
    app: bookinfo-dev-ratings
```

* Create `ratings-ingress.yaml` file inside `k8s/` directory with below content

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: bookinfo-dev-ratings
  namespace: student[X]-bookinfo-dev
spec:
  rules:
  - host: bookinfo.dev.opsta.net
    http:
      paths:
      - path: /student[X]/ratings(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: bookinfo-dev-ratings
            port:
              number: 8080
```

```bash
# Create deployment resource
kubectl apply -f k8s/

# Check status of each resource
kubectl get deploy,pod,svc,ingress
```

* Try to access <https://bookinfo.dev.opsta.net/student[X]/ratings/health> and <https://bookinfo.dev.opsta.net/student[X]/ratings/ratings/1> to check the deployment

Next: [Deploy MongoDB with Helm Chart](09-helm-mongodb.md)
