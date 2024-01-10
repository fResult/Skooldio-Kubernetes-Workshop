# Deploy MongoDB with Helm Chart

## Add Helm Bitnami Charts

```bash
# Add bitnami charts repository
helm repo add bitnami https://charts.bitnami.com/bitnami
# List repo
helm repo list
# Update repo
helm repo update
```

## Deploy basic MongoDB with Helm Charts from Kubeapps Hub

* Go to <https://github.com/bitnami/charts/tree/master/bitnami/mongodb> to see MongoDB Helm Charts description
* Deploy basic MongoDB

```bash
# Check helm release
helm list
# Deploy MongoDB Helm Charts
helm install mongodb bitnami/mongodb
# Check helm release again
helm list
```

* Check if MongoDB is working

```bash
# Check resources
kubectl get pod
kubectl get deployment
kubectl get service
```

* Connect to MongoDB

```bash
# Check secret
kubectl get secret
kubectl describe secret mongodb

# Get MongoDB root password
export MONGODB_ROOT_PASSWORD=$(kubectl get secret mongodb \
  -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)
# Create another pod to connect to MongoDB via mongo cli command
kubectl run mongodb-client --rm --tty -i --restart='Never' --image bitnami/mongodb:4.4.4-debian-10-r27 \
  --command -- mongo admin --host mongodb --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD
show dbs
exit
```

## Deploy custom configuration MongoDB with Helm Value

### Create Kubernetes Secret for MongoDB credential

* Create folder to store secret with command `mkdir ~/bookinfo-secret` and create file `touch ~/bookinfo-secret/bookinfo-dev-ratings-mongodb-secret.yaml`
* This folder normally will be created and used by administrator of Kubernetes and developer can not access

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: bookinfo-dev-ratings-mongodb-secret
  namespace: bookinfo-dev
type: Opaque
data:
  mongodb-root-password: "cmF0aW5ncy1kZXYtcm9vdA=="
  mongodb-password: "cmF0aW5ncy1kZXY="
```

* Create secret with following commands

```bash
kubectl apply -f k8s/bookinfo-secret/bookinfo-dev-ratings-mongodb-secret.yaml
kubectl get secret
kubectl describe secret bookinfo-dev-ratings-mongodb-secret
echo $(kubectl get secret bookinfo-dev-ratings-mongodb-secret \
  -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)
echo $(kubectl get secret bookinfo-dev-ratings-mongodb-secret \
  -o jsonpath="{.data.mongodb-password}" | base64 --decode)
```

* For the case we want to encode
```bash
echo -n ratings-dev-root | base64
echo -n ratings-dev | base64
```

### Prepare Helm Value file

* Check `values.yaml` on <https://github.com/bitnami/charts/tree/master/bitnami/mongodb> to see all configuration
* `mkdir ~/ratings/k8s/helm-values/` and create `values-bookinfo-dev-ratings-mongodb.yaml` file inside `helm-values` directory and put below content

```yaml
image:
  tag: 4.4.4-debian-10-r27
auth:
  existingSecret: bookinfo-dev-ratings-mongodb-secret
  username: ratings-dev
  database: ratings-dev
persistence:
  enabled: false
initdbScriptsConfigMap: bookinfo-dev-ratings-mongodb-initdb
```

### Create initial script with Config Map

* Create `databases` directory inside ratings directory with command `mkdir databases/`
* Create `ratings_data.json` inside `databases` directory with following content

```json
{"rating": 5}
{"rating": 4}
```

* Create `ratings_data.json` inside `databases` directory with following content

```bash
#!/bin/sh

set -e
mongoimport --host localhost --username $MONGODB_USERNAME --password $MONGODB_PASSWORD \
  --db $MONGODB_DATABASE --collection ratings --drop --file /docker-entrypoint-initdb.d/ratings_data.json
```
> Note:
> How can we know `$MONGODB_USERNAME`, `$MONGODB_PASSWORD`, and `$MONGODB_DATABASE` are able to use?
>> Because of [HelmChart of Bitnami/Mongodb](https://github.com/bitnami/charts/tree/master/bitnami/mongodb) has `auth.username`, `auth.password`, and `auth.database`.\
>> And the [DockerHub of Bitnami/Mongodb](https://hub.docker.com/r/bitnami/mongodb) in the `Creating a user and database on first run` topic has `$MONGODB_USERNAME`, `$MONGODB_PASSWORD`, and `$MONGODB_DATABASE`.\
>> And the *HelmChart of Bitnami/Mongodb* also use this *Bitnami/Mongodb* in DockerHub.
>>> So, that means we can use these three variables by default.\
> How can we know that `ratings_data.json` is in `/docker-entrypoint-initdb.d`?
>> In the [DockerHub of Bitnami/Mongodb](https://hub.docker.com/r/bitnami/mongodb) at the `Initializing a new instance` topic has explained detail.\
>> And we can also use `$MONGODB_INITSCRIPTS_DIR` variable.

* Run following commands to create configmap

```bash
# Create configmap
kubectl create configmap bookinfo-dev-ratings-mongodb-initdb \
  --from-file=databases/ratings_data.json \
  --from-file=databases/script.sh

# Check config map
kubectl get configmap
kubectl describe configmap bookinfo-dev-ratings-mongodb-initdb
```

### Deploy MongoDB with Helm Value file

```bash
# Delete first mongodb release
helm list
helm delete mongodb

# Deploy new MongoDB with Helm Value
helm install -f k8s/helm-values/values-bookinfo-dev-ratings-mongodb.yaml \
  bookinfo-dev-ratings-mongodb bitnami/mongodb
```

* Check if MongoDB is working properly with imported data

```bash
# Get MongoDB root password
export MONGODB_ROOT_PASSWORD=$(kubectl get secret bookinfo-dev-ratings-mongodb-secret \
  -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)
# Create another pod to connect to MongoDB via mongo cli command
kubectl run mongodb-client --rm --tty -i --restart='Never' --image bitnami/mongodb:4.4.4-debian-10-r27 \
  --command -- mongo admin --host bookinfo-dev-ratings-mongodb --authenticationDatabase admin \
  -u root -p $MONGODB_ROOT_PASSWORD
show dbs
use ratings-dev
show collections
db.ratings.find()
exit
```

## Update Ratings Service Manifest File

* Update Ratings Service Manifest File to read data from MongoDB in `~/ratings/k8s/ratings-deployment.yaml` file

```yaml
...
        env:
        - name: SERVICE_VERSION
          value: v2
        - name: MONGO_DB_URL
          value: mongodb://bookinfo-dev-ratings-mongodb:27017/ratings-dev
        - name: MONGO_DB_USERNAME
          value: ratings-dev
        - name: MONGO_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: bookinfo-dev-ratings-mongodb-secret
              key: mongodb-password
...
```

* Run `kubectl apply -f ~/ratings/k8s/` and test if ratings service still working

Next: [Convert Rating Service to Helm](06-helm-rating.md)
