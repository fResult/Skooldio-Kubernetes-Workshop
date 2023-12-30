# Kubernetes Command Line Workshop

## Getting started

```bash
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
kubectl create namespace bookinfo-dev
kubectl create namespace bookinfo-uat
kubectl create namespace bookinfo-prd
# Show your newly created namespace
kubectl get namespaces
# Set default namespace
kubectl config set-context $(kubectl config current-context) --namespace=bookinfo-dev
kubectl config get-contexts
```

## Create Pod, Deployment and Service

```bash
# Create nginx deployment
kubectl create deployment nginx --image=nginx
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
kubectl get deployment,pod
# Change nginx deployment to use apache instead
kubectl set image deployment nginx nginx=httpd:2.4-alpine --record
# See change
watch -n1 kubectl get pod
kubectl get deployment
kubectl describe deployments nginx
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
# Scale apache deployment to 3 replicas
kubectl scale deployment apache --replicas=3
# See change
kubectl get deployment,pod
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
kubectl get service
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
kubectl get deployment,pod
```

Next: [Kubernetes Manifest Files Workshop](03-k8s-manifest.md)
