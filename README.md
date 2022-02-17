# Redis-Enterprise-on-k8s-the-easy-way

This guide would help you to setup Kubernetes Cluster and Redis Enterprise Cluster with three nodes, using Redis Enterprise Kuberneted Operator

## Provision the k8s cluster

Choose one of the links below for instructions on provisioning the managed kubernetes cluster in the cloud.

Note: you need to have appropriate priveledges to provision cluster.

Note #2: these clusters would incur cost. Make sure to delete the clusters after you are done with them. Each instruction includes section on cleaning up the cluster.

- [Google GKE](GoogleGKE.md) time required ~ 7 minutes
- [Amazon EKS](AmazonEKS.md) time required ~ 25 minutes
- [Azure AKS](AzureAKS.md) time required ~ 7 minutes

Make sure to use node types with at least 4 cores and 16GB RAM.

## Install Redis Enterprise Operator

Time reqired - less then 1 minute

```bash
kubectl apply -f https://raw.githubusercontent.com/RedisLabs/redis-enterprise-k8s-docs/master/bundle.yaml
```

Ref: https://github.com/RedisLabs/redis-enterprise-k8s-docs

## Create 3 node REC cluster

Time reqired ~ 10 minutes

```bash
kubectl apply -f - << EOF   
apiVersion: app.redislabs.com/v1
kind: RedisEnterpriseCluster
metadata:
  name: rec
spec:
  # Add fields here
  nodes: 3
EOF
```

Optionally you might specify extended Memory and CPU requests for the cluster. Make sure your node type can accomodate it.

With explisit (large) CPU/Memory spec
```bash
kubectl apply -f - << EOF                                              
apiVersion: app.redislabs.com/v1
kind: RedisEnterpriseCluster
metadata:
  name: rec
spec:
  nodes: 3
  redisEnterpriseNodeResources:
    limits:
      cpu: 12000m
      memory: 24Gi
    requests:
      cpu: 12000m
      memory: 24Gi
EOF
```

kubectl command returns almost immediatly, but the actual cluster creation time might take 5-10 minutes and more with larger # of nodes.

## Retreive REC username/password
```bash
REC_UNAME=$(kubectl get secret rec -o jsonpath='{.data.username}' | base64 --decode)
echo $REC_UNAME
REC_PASSWORD=$(kubectl get secret rec -o jsonpath='{.data.password}' | base64 --decode)
echo $REC_PASSWORD
```

## Port forwarding

The easiest way to access cluster UI and API from your local machine is port forwarding.

Execute these commands in separete terminal sessions and keep these session open.

### for cluster UI
```bash
kubectl port-forward service/rec-ui 8443:8443
```
then go to https://localhost:8443/ ignore the certificate warning and proceed with the username/password retreived on a previouse step.

### For REST API
```bash
kubectl port-forward rec-0 9443:9443
```

Test cluster API access/retreive cluster nodes
```bash
curl -X GET https://localhost:9443/v1/nodes --insecure \
-H "content-type: application/json" \
-u "$REC_UNAME:$REC_PASSWORD"
```

## Connect to the node
```bash
kubectl exec -it rec-0 -- bash
rladmin status
```

## Create database
```bash
kubectl apply -f - << EOF                                              
apiVersion: app.redislabs.com/v1alpha1
kind: RedisEnterpriseDatabase
metadata:
  name: redis-enterprise-database
spec:
  redisEnterpriseCluster:
    name: rec
EOF
```

## Update database with replicatin and shards=2
```bash
kubectl apply -f - << EOF                                              
apiVersion: app.redislabs.com/v1alpha1
kind: RedisEnterpriseDatabase
metadata:
  name: redis-enterprise-database
spec:
  redisEnterpriseCluster:
    name: rec
  replication: true
  shardCount: 2
  memorySize: 1GB
EOF
```


## Troubleshooting

### Cloud account limit exceeded

k8s Cluster creation fails with messages like:
- UnsupportedAvailabilityZoneException
- Number of Cores per region limit exceeded
- Can not create VPC/limit exceedd

You are reaching one of your account service limits. Try deleting unused resources such as VPCs or VMs, provision cluster in a different region, request service limit/quota increase.

### Rec pod stuck in Pending state

Likely insuficcient memory/CPU resources on the nodes. VM type of the node is too small for REC cluster
```
kubectl get pods
NAME                                         READY   STATUS    RESTARTS   AGE
rec-0                                        0/2     Pending   0          22s
rec-services-rigger-5965fd4cbf-6mkpg         1/1     Running   0          22s
redis-enterprise-operator-7f8d8548c5-b8n7g   2/2     Running   0          66s

kubectl describe pod rec-0
...
Events:
  Type     Reason            Age                From               Message
  ----     ------            ----               ----               -------
  Warning  FailedScheduling  32s (x2 over 33s)  default-scheduler  0/3 nodes are available: 2 Insufficient memory, 3 Insufficient cpu.
```
Increase node size, decrease limits.

### Insufficient permissions

Messages like User “abc@redis.com” cannot create resource “roles” in API group “rbac.authorization.k8s.io” in the namespace “default”: requires one of [“container.roles.create”] permission(s).

Reach out to your cloud account administrator to get sufficient permissions.