# Google Kubernetes Engine

Setup time ~ 7 minutes

## Install Google Cloud CLI

Ref: https://cloud.google.com/sdk/docs/install

## login 
```
gcloud init
```

## create cluster
```
gcloud container clusters create my-redis-cluster \
  --num-nodes 3 --machine-type e2-standard-4
```
Ref: gcloud container clusters create documentation https://cloud.google.com/sdk/gcloud/reference/container/clusters/create 

Ref: VM types/sizes: https://cloud.google.com/compute/docs/general-purpose-machines#e2-standard recommended node size - at least 4 cores/16GB RAM

## add cluster credentials to kubectl

`gcloud container clusters create` automatically adds newly created cluster credentials as the default context upon creation. No additional actions needed.

## delete cluster
```
gcloud container clusters delete my-redis-cluster 
```

## Clear up kubectl context
```
kubectl config delete-context my-redis-cluster
```