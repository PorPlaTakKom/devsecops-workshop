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
  type: ClusterIP
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

Next: [Static Website on Kubernetes Workshop](docs/07-k8s-static-web.md)
