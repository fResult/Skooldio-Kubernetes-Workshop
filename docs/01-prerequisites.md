# Prerequisites and Preparation before Workshop

## Prerequisites

We recommend you to register for Google Cloud Platform account to run through this workshop.

* Text editor
  * Google Cloud Shell Editor (recommend)
  * VSCode
  * VIM
* For Linux Terminal in this workshop you can use
  * Google Cloud Shell (recommend)
  * Linux Terminal
  * MacOS Terminal
  * Windows Subsystem Linux 2 (WSL2)
* Docker Registry with private access
  * GCR: Google Cloud Container Registry (recommend)
  * ECR: Amazon Elastic Container Registry
  * Azure Container Registry
  * Docker Hub (free 1 private registry)
* [Install docker command](https://docs.docker.com/engine/install/)
* [Install kubectl command](https://kubernetes.io/docs/tasks/tools/)
* [Install helm command](https://helm.sh/docs/intro/install/)
* Kubernetes Cluster on Public Cloud
  * GKE: Google Kubernetes Engine (recommend)
  * EKS: Amazon Elastic Kubernetes Service
  * AKS: Azure Kubernetes Service
* kubeconfig with create and full privilege control on namespaces
* 3 domains ready to point to bookinfo application for each environment. For example
  * Dev Environment: `bookinfo.dev.opsta.net`
  * UAT Environment: `bookinfo.uat.opsta.net`
  * Production Environment: `bookinfo.opsta.net`

## Preparation

This is how we prepare Kubernetes Workshop on Google Cloud Platform. We running all the commands on Google Cloud Shell

### Prepare Bookinfo Docker Image on Google Cloud Container Registry (GCR)

```bash
# CHANGE THESE VARIABLES
export K8S_NAME=skooldio-k8s
export K8S_ZONE=asia-southeast1-b
export PROJECT_ID=zcloud-cicd
gcloud config set project $PROJECT_ID

# Authentication with GCR
gcloud iam service-accounts create $K8S_NAME
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member "serviceAccount:$K8S_NAME@$PROJECT_ID.iam.gserviceaccount.com" --role "roles/storage.admin"
gcloud iam service-accounts keys create keyfile.json --iam-account $K8S_NAME@$PROJECT_ID.iam.gserviceaccount.com
cat keyfile.json | docker login -u _json_key --password-stdin https://asia.gcr.io

# Pull all bookinfo micro-services
docker pull opsta/bookinfo-ratings:latest
docker pull opsta/bookinfo-productpage:latest
docker pull opsta/bookinfo-details:latest
docker pull opsta/bookinfo-reviews:latest

# Tag to your GCR account
docker tag opsta/bookinfo-ratings:latest asia.gcr.io/$PROJECT_ID/bookinfo-ratings:dev
docker tag opsta/bookinfo-ratings:latest asia.gcr.io/$PROJECT_ID/bookinfo-ratings:uat
docker tag opsta/bookinfo-ratings:latest asia.gcr.io/$PROJECT_ID/bookinfo-ratings:prd
docker tag opsta/bookinfo-productpage:latest asia.gcr.io/$PROJECT_ID/bookinfo-productpage:dev
docker tag opsta/bookinfo-productpage:latest asia.gcr.io/$PROJECT_ID/bookinfo-productpage:uat
docker tag opsta/bookinfo-productpage:latest asia.gcr.io/$PROJECT_ID/bookinfo-productpage:prd
docker tag opsta/bookinfo-details:latest asia.gcr.io/$PROJECT_ID/bookinfo-details:dev
docker tag opsta/bookinfo-details:latest asia.gcr.io/$PROJECT_ID/bookinfo-details:uat
docker tag opsta/bookinfo-details:latest asia.gcr.io/$PROJECT_ID/bookinfo-details:prd
docker tag opsta/bookinfo-reviews:latest asia.gcr.io/$PROJECT_ID/bookinfo-reviews:dev
docker tag opsta/bookinfo-reviews:latest asia.gcr.io/$PROJECT_ID/bookinfo-reviews:uat
docker tag opsta/bookinfo-reviews:latest asia.gcr.io/$PROJECT_ID/bookinfo-reviews:prd

# Push docker image back to your GCR account
docker push asia.gcr.io/$PROJECT_ID/bookinfo-ratings:dev
docker push asia.gcr.io/$PROJECT_ID/bookinfo-ratings:uat
docker push asia.gcr.io/$PROJECT_ID/bookinfo-ratings:prd
docker push asia.gcr.io/$PROJECT_ID/bookinfo-productpage:dev
docker push asia.gcr.io/$PROJECT_ID/bookinfo-productpage:uat
docker push asia.gcr.io/$PROJECT_ID/bookinfo-productpage:prd
docker push asia.gcr.io/$PROJECT_ID/bookinfo-details:dev
docker push asia.gcr.io/$PROJECT_ID/bookinfo-details:uat
docker push asia.gcr.io/$PROJECT_ID/bookinfo-details:prd
docker push asia.gcr.io/$PROJECT_ID/bookinfo-reviews:dev
docker push asia.gcr.io/$PROJECT_ID/bookinfo-reviews:uat
docker push asia.gcr.io/$PROJECT_ID/bookinfo-reviews:prd
```

### Create Kubernetes Cluster on Google Cloud

```bash
# Create GKE Cluster
gcloud container --project "$PROJECT_ID" clusters create "$K8S_NAME" --zone "$K8S_ZONE" \
  --cluster-version "1.20.2-gke.2500" --release-channel "rapid" --machine-type "e2-medium" \
  --enable-ip-alias --image-type "COS_CONTAINERD" --disk-size "100" --num-nodes "2" \
  --network "default" --subnetwork "default" --preemptible

# Get kubeconfig from GKE
gcloud container clusters get-credentials $K8S_NAME --project $PROJECT_ID --zone $K8S_ZONE
```

### Prepare ingress controller and map domain

```bash
# Add Nginx Helm Repesitory
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Deploy Nginx Controller on GKE
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace kube-system

# You need to wait for few minutes before you will get the EXTERNAL-IP
# Get Controller IP Address
echo $(kubectl --namespace kube-system get services ingress-nginx-controller \
  --output jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

* Map your domain with IP Address above

Next: [Kubernetes Commands Workshop](02-k8s-cli.md)
