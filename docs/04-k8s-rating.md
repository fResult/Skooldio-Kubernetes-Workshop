# Deploy Bookinfo Rating Service on Kubernetes

## Create Secret to pull Docker Image from Nexus Docker Private Registry

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

* `mkdir -p ~/ratings/k8s/` to make a directory to store manifest file
* Create `ratings-deployment.yaml` file inside `~/ratings/k8s/` directory with below content

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bookinfo-dev-ratings
  namespace: bookinfo-dev
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
        image: asia.gcr.io/[PROJECT_ID]/bookinfo-ratings:dev
        imagePullPolicy: Always
        ports:
        - containerPort: 9080
          name: web-port
          protocol: TCP
        livenessProbe:
          httpGet:
            path: /health
            port: 9080
            scheme: HTTP
        readinessProbe:
          httpGet:
            path: /health
            port: 9080
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
  namespace: bookinfo-dev
spec:
  type: ClusterIP
  ports:
  - port: 9080
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
  namespace: bookinfo-dev
spec:
  rules:
  - host: bookinfo.dev.opsta.net
    http:
      paths:
      - path: /ratings(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: bookinfo-dev-ratings
            port:
              number: 9080
```

```bash
# Create deployment resource
kubectl apply -f k8s/

# Check status of each resource
kubectl get deployment
kubectl get service
kubectl get ingress
```

* Try to access <https://bookinfo.dev.opsta.net/ratings/health> and <https://bookinfo.dev.opsta.net/ratings/ratings/1> to check the deployment

Next: [Deploy MongoDB with Helm Chart](05-helm-mongodb.md)
