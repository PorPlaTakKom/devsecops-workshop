# Kubernetes Commands Workshop

## Prerequisites

* Linux Terminal, Google Cloud Shell, MacOS Terminal, or WSL2 on Windows
* kubectl command
* Your own Kubernetes Cluster
* kubeconfig with create and full privilege control on namespaces
* To play with service type Load Balancer
  * Use Kubernetes on any public cloud provider such as GKE, EKS, or AKS
  * Use Kubernetes on-premise with metallb
* Your own text editor such as Vim or VSCode

## Getting started

```bash
# Get kubeconfig from GKE
gcloud container clusters get-credentials k8s --project zcloud-cicd --zone asia-southeast1-a
# Check if it can connect to cluster
kubectl version
# View Cluster Information
kubectl cluster-info
# Show all pods
kubectl get pod --all-namespaces
```

## Setup your own namespaces

```bash
# Show all namespaces
kubectl get namespaces
# Show current cluster connection configuration
kubectl config get-contexts
# Create your own namespaces
kubectl create namespace student[X]-bookinfo-dev
kubectl create namespace student[X]-bookinfo-uat
kubectl create namespace student[X]-bookinfo-prd
# Show your newly created namespace
kubectl get namespaces
# Set default namespace
kubectl config set-context $(kubectl config current-context) --namespace=student[X]-bookinfo-dev
kubectl config get-contexts
```

## Create Pod, Deployment and Service

```bash
# Create nginx deployment
kubectl create deployment nginx --image=nginx
# Set command history for record history
kubectl annotate deployment nginx kubernetes.io/change-cause='initial deployment'
# Show pods
kubectl get pods
# Show deployments
kubectl get deployment
# Show nginx deployment detail
kubectl describe deployment nginx
# Expose service load balancer to nginx deployment port 80
kubectl expose deployment nginx --type LoadBalancer --port 80
# Wait to see public ip to be active and test it
kubectl get services -w
```

## Scale service and change docker image

```bash
# Scale pod to 3 replicas
kubectl scale deployment nginx --replicas=3
# Show deployment and pod status
kubectl get deployment
kubectl get pod
# Change nginx deployment to use apache instead
kubectl set image deployment nginx nginx=httpd:2.4-alpine --record
# See change
watch -n1 kubectl get pod
kubectl get deployment
kubectl describe deployments nginx
```

## Use Service ClusterIP and access by Proxy

```bash
kubectl expose deployment nginx --type ClusterIP --port 80
kubectl get service
kubectl proxy --port=8080
# Click preview port 8080
# access via /api/v1/namespaces/student[xxx]-bookinfo-dev/services/nginx:80/proxy/
```

## Rollback deployment

```bash
# Show deployment history
kubectl rollout history deployment nginx
# Rollback one version
kubectl rollout undo deployment nginx
# See change
kubectl rollout history deployment nginx
kubectl describe deployment nginx
```

## Label and Selector

```bash
# Create new apache deployment
kubectl create deployment apache --image=httpd:2.4-alpine
# Set command history for record history
kubectl annotate deployment apache kubernetes.io/change-cause='initial apache deployment'
# Scale apache deployment to 3 replicas
kubectl scale deployment apache --replicas=3
# See change
kubectl get pod
kubectl get deployment
# See the label and selector
kubectl describe deployments nginx
kubectl describe service nginx
# See the label
kubectl describe deployments apache
# Set service nginx to select apache deployment label instead
kubectl set selector service nginx 'app=apache'
# Revert selector back
kubectl set selector service nginx 'app=nginx'
```

## Kubernetes Utilities Commands

```bash
# Show pod log
kubectl get pod
kubectl logs apache-5d94cf486d-65m4b -f
# Enter inside container
kubectl exec -it apache-5d94cf486d-65m4b -- sh
ping nginx
exit
# View node information
kubectl get nodes
kubectl describe nodes gke-training-default-pool-115e6de5-shkw
```

## Clear Everything

```bash
kubectl delete deployment nginx apache
kubectl delete service nginx
kubectl get deployments
kubectl get pod
```

Next: [Kubernetes Manifest File Workshop](06-k8s-manifest.md)
