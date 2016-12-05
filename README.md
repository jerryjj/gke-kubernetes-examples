# Google Container Engine - demo

This is really simple collection of Google Container engine demos for quick
hands-on experiments on different concepts...
Use as you want :)

Before deploying any of the Pods, replace the Project Id placeholder with your
own project id.

## Cluster
gcloud beta container image describe
gcloud container builds


```sh
export PROJECT_ID=[REPLACE-WITH_OWN-PROJECT_ID]
gcloud config set project $PROJECT_ID

gcloud container clusters create gkedemo \
--zone europe-west1-c --machine-type "n1-standard-2" \
--image-type "GCI" --disk-size "100" \
--num-nodes 2 --enable-cloud-logging

export CLUSTER_CONTEXT=$(kubectl config view | awk '{print $2}' | grep "gkedemo" | tail -n 1)
```

## Container Registry and Container builder

To build and upload locally run:

```sh
cd container/
docker build -t eu.gcr.io/$PROJECT_ID/hello-node:v1 .
gcloud docker -- push eu.gcr.io/$PROJECT_ID/hello-node:v1
```

To build in the cloud:

```sh
cd container/
gcloud container builds submit --tag eu.gcr.io/$PROJECT_ID/hello-node:v2 .

gcloud container builds describe [REPLACE-WITH-BUILD_ID]
```

## ConfigMaps

Create Configs:

```sh
kubectl --context=$CLUSTER_CONTEXT create configmap fileconfig --from-file=./configs
kubectl --context=$CLUSTER_CONTEXT create -f schemas/manualconfig.yaml
```

Inspect Configs:

```sh
kubectl --context=$CLUSTER_CONTEXT get configmaps
kubectl --context=$CLUSTER_CONTEXT describe configmaps/fileconfig
kubectl --context=$CLUSTER_CONTEXT get configmaps/fileconfig -o yaml
```

Test configs on pods:

```sh
kubectl --context=$CLUSTER_CONTEXT create -f schemas/manconf-env-pod.yaml
kubectl --context=$CLUSTER_CONTEXT logs manconf-env-pod | grep "CFG_"

kubectl --context=$CLUSTER_CONTEXT create -f schemas/manconf-fs-pod.yaml
kubectl --context=$CLUSTER_CONTEXT logs manconf-fs-pod

kubectl --context=$CLUSTER_CONTEXT create -f schemas/fileconf-pod.yaml
kubectl --context=$CLUSTER_CONTEXT exec -it fileconf-pod bash
kubectl --context=$CLUSTER_CONTEXT delete pods/fileconf-pod
```

## Secrets

Creating secrets:

```sh
kubectl --context=$CLUSTER_CONTEXT create secret generic db-pass --from-file=./secrets/database/password
```

Inspecting secrets:

```sh
kubectl --context=$CLUSTER_CONTEXT get secrets
kubectl --context=$CLUSTER_CONTEXT describe secrets/db-pass
kubectl --context=$CLUSTER_CONTEXT get secrets/db-pass -o yaml
echo "cmVhbGx5LXNlY3JldC1kYi1wYXNzd29yZAo=" | base64 -D
```

Test secrets on pods:

```sh
kubectl --context=$CLUSTER_CONTEXT create -f schemas/secret-pod.yaml
kubectl --context=$CLUSTER_CONTEXT exec -it secret-pod bash
kubectl --context=$CLUSTER_CONTEXT delete pods/secret-pod
```

## Deployments

```sh
kubectl --context=$CLUSTER_CONTEXT create -f schemas/hello-deployment.yaml
kubectl --context=$CLUSTER_CONTEXT get deployments
kubectl --context=$CLUSTER_CONTEXT get pods

kubectl --context=$CLUSTER_CONTEXT create -f schemas/hello-service.yaml
kubectl --context=$CLUSTER_CONTEXT get services
```

Inspecting through proxy:

```sh
kubectl --context=$CLUSTER_CONTEXT proxy
```

Rolling update:

```sh
kubectl --context $CLUSTER_CONTEXT set image deployment/hello-deployment hello=eu.gcr.io/$PROJECT_ID/hello-node:v2
kubectl --context $CLUSTER_CONTEXT rollout status deployment/hello-deployment

kubectl --context $CLUSTER_CONTEXT rollout history deployment/hello-deployment
kubectl --context $CLUSTER_CONTEXT rollout history deployment/hello-deployment --revision=1
kubectl --context $CLUSTER_CONTEXT rollout undo deployment/hello-deployment

kubectl --context $CLUSTER_CONTEXT describe deployment
kubectl --context $CLUSTER_CONTEXT get rs
kubectl --context $CLUSTER_CONTEXT describe rs/hello-deployment-

kubectl --context $CLUSTER_CONTEXT rollout undo deployment/hello-deployment --to-revision=2
```

## Ingress

```sh
kubectl --context=$CLUSTER_CONTEXT create -f schemas/hello-ing-service.yaml
kubectl --context=$CLUSTER_CONTEXT create -f schemas/hello-ingress.yaml
kubectl --context=$CLUSTER_CONTEXT describe ingress/hello-ingress
```
