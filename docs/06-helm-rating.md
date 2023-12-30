# Convert Rating Service to Helm

## Create Helm Chart for Ratings Service

* Delete current Ratings Service first with command `kubectl delete -f k8s/`
* `mkdir ~/ratings/k8s/helm` to create directory for Ratings Helm Charts
* Create `Chart.yaml` file inside `helm` directory and put below content

```yaml
apiVersion: v1
description: Bookinfo Ratings Service Helm Chart
name: bookinfo-ratings
version: 1.0.0
appVersion: 1.0.0
home: https://bookinfo.demo.opsta.net/ratings
maintainers:
  - name: Developer
    email: skooldio@opsta.net
```

* `mkdir ~/ratings/k8s/helm/templates` to create directory for Helm Templates
* Move our ratings manifest file to template directory with command `mv k8s/ratings-*.yaml k8s/helm/templates/`
* Let's try deploy Ratings Service

```bash
# Deploy Ratings Helm Chart
helm install bookinfo-dev-ratings k8s/helm

# Get Status
kubectl get deployment
kubectl get pod
kubectl get service
kubectl get ingress
```

* Try to access <https://bookinfo.dev.opsta.net/ratings/health> and <https://bookinfo.dev.opsta.net/ratings/ratings/1> to check the deployment

## Create Helm Value file for Ratings Service

* Create `values-bookinfo-dev-ratings.yaml` file inside `k8s/helm-values` directory and put below content

```yaml
ratings:
  namespace: bookinfo-dev
  image: asia.gcr.io/[PROJECT_ID]/bookinfo-ratings
  tag: dev
  replicas: 1
  imagePullSecrets: registry-bookinfo
  port: 9080
  healthCheckPath: "/health"
  mongodbPasswordExistingSecret: bookinfo-dev-ratings-mongodb-secret
ingress:
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  host: bookinfo.dev.opsta.net
  path: "/ratings(/|$)(.*)"
  serviceType: ClusterIP
extraEnv:
  SERVICE_VERSION: v2
  MONGO_DB_URL: mongodb://bookinfo-dev-ratings-mongodb:27017/ratings-dev
  MONGO_DB_USERNAME: ratings-dev
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
        {{- if .Values.ratings.mongodbPasswordExistingSecret }}
        - name: MONGO_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ .Values.ratings.mongodbPasswordExistingSecret }}
              key: mongodb-password
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

## Exercise: Deploy on UAT and Production Environment

* Create Helm value and deploy for UAT and Production environment
* Create Kubernetes & Helm deployment for `details`, `reviews`, and `productpage` services

### Hints

* Prepare Helm Values for mongodb and ratings
* Change namespace (context) to uat environment
* Add configmap and secret
* Helm install mongodb release
* Helm install ratings release
