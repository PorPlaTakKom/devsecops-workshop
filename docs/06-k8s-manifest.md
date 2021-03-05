# Kubernetes Manifest Files Workshop

## Prerequisites

* Linux Terminal, Google Cloud Shell, MacOS Terminal, or WSL2 on Windows
* Docker
* Docker Compose
* Docker Registry such as Docker Hub, Nexus, GitLab Docker Registry, GitHub Docker Registry, or JFrog
* kubectl command
* Your own Kubernetes Cluster
* kubeconfig with create and full privilege control on namespaces
* To play with service type Ingress, you need to deploy Nginx Ingress Controller and map external ip address with domain
* Your own text editor such as Vim or VSCode

## Deploy Pod with Manifest File

* `mkdir ~/k8s` to create folder for Kubernetes Manifest File
* Create `01-pod.yaml` file with below content

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: student[X]-bookinfo-dev
spec:
  containers:
  - name: busybox
    image: busybox
    command:
    - sleep
    - "3600"
```

* Create pod from manifest file

```bash
cd ~/k8s
# Create resources as configured in manifest file
kubectl apply -f 01-pod.yaml
kubectl get pod

# Try to get inside pod
kubectl exec -it busybox -- sh
ping www.google.com
exit
```

## How to check syntax for manifest file

```bash
# Show all api resources in Kubernetes Cluster
kubectl api-resources
# Show manifest syntax for Kind = pod
kubectl explain pod
# Show all manifest syntax for Kind = pod
kubectl explain pod --recursive
# Show manifest spec syntax for Kind = pod
kubectl explain pod.spec
# Show manifest syntax for Kind = deployment
kubectl explain deployment
```

## Deployment and Service Manifest File

* Create `02-apache-deployment.yaml` file with below content

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache
  namespace: student[X]-bookinfo-dev
  labels:
    app: apache
spec:
  replicas: 3
  selector:
    matchLabels:
      app: apache
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - image: httpd:2.4.43-alpine
        name: apache
```

* Create `02-apache-service.yaml` file with below content

```yaml
apiVersion: v1
kind: Service
metadata:
  name: apache
  namespace: student[X]-bookinfo-dev
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: apache
```

```bash
kubectl create -f 02-apache-deployment.yaml -f 02-apache-service.yaml --record
kubectl get deployments
kubectl get services -w
```

## Clean every deployment and service

```bash
# Show all deployments
kubectl get deployment
# Delete all deployments
kubectl delete deployment apache
# Check if any deployment left
kubectl get deployment

# Show all services
kubectl get service
# Delete all services
kubectl delete service apache
# Check if any service left
kubectl get service

# Show all pods
kubectl get pod
# Delete all pods
kubectl delete pod busybox
# Check if any pods left
kubectl get pod
```

## Deploy Ratings service on Kubernetes

### Prepare GitHub personal access tokens

* Go to <https://github.com/settings/tokens> or Profile > Settings > Developer settings > Personal access tokens
* Click on `Generate new token`
* Maybe GitHub will ask for your password
* Create new personal access token
  * Note: `int209`
  * Select scopes
    * all `repo`
    * all `write:packages`
    * all `delete:packages`
* Copy token or else YOUR WILL LOSE IT FOREVER!

### Prepare Rating Service Docker Image First

```bash
cd ~/ratings
docker-compose up --build
```

* Test your application to make sure it really works
* Ctrl+C to exit from Docker Compose and put `docker-compose down` to delete stopped container

### Push Rating Service Docker Image to Nexus

```bash
# Login to Private Registry first
docker login ghcr.io
# Put your username and token above

# Check to see Rating Service Docker Image name
docker images
# Push Rating Docker Image
docker push ghcr.io/[GITHUB_USER]/bookinfo-ratings:dev
```

* Check your pushed Docker Image at <https://nexus.demo.opsta.co.th/#browse/browse:docker-registry-private>

### Create Secret to pull Docker Image from Nexus Docker Private Registry

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

### Create Rating Service Kubernetes Manifest File

* `mkdir ~/ratings/k8s/` to make a directory to store manifest file
* Create `ratings-deployment.yaml` file inside `~/ratings/k8s/` directory with below content

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
        image: registry.demo.opsta.co.th/student[X]/bookinfo/ratings:dev
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

* Create `ratings-service.yaml` file inside `~/ratings/k8s/` directory with below content

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

* Create `ratings-ingress.yaml` file inside `~/ratings/k8s/` directory with below content

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
  - host: bookinfo.dev.opsta.co.th
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
kubectl get deployment
kubectl get service
kubectl get ingress
```

* Try to access <https://bookinfo.dev.opsta.co.th/student[X]/ratings/health> and <https://bookinfo.dev.opsta.co.th/student[X]/ratings/ratings/1> to check the deployment
* Commit and push your code
