# boundary-kubernetes
Deploy HashiCorp Boundary to a Kubernetes cluster.

## Deployment Instructions
Makefile:
```bash
$ make boundary-deploy
```

Manual method:
```bash
# Create opaque secret containing PostgreSQL credentials
$ kubectl create secret generic psql-creds \
  --from-literal=username=postgres \
  --from-literal=password=postgres \
  --from-literal=database=boundary
secret/psql-creds created

# Create the dedicated service account for Boundary
$ kubectl apply \
  -f boundary-serviceaccount.yaml
serviceaccount/boundary-sa created

# Deploy PostgreSQL
$ kubectl apply \
  -f manifests/postgresql-statefulset.yaml \
  -f manifests/postgresql-service.yaml
statefulset.apps/postgresql created
service/postgresql created

# Deploy Configmaps for Boundary Controller and Worker
$ kubectl apply \
  -f manifests/controller-config-configmap.yaml \
  -f manifests/worker-config-configmap.yaml
configmap/controller-config created
configmap/worker-config created

# Deploy the Kubernetes job to initalize the PostgreSQL database
$ kubectl apply \
  -f manifests/database-init-job.yaml
job.batch/database-init created

# Deploy Boundary Controller daemonset and services
$ kubectl apply \
  -f manifests/controller-daemonset.yaml \
  -f manifests/controller-api-service.yaml \
  -f manifests/controller-cluster-service.yaml
daemonset.apps/boundary-controller created
service/controller-api created
service/controller-cluster created

# Deploy Boundary Worker daemonset and services
$ kubectl apply \
  -f manifests/worker-daemonset.yaml \
  -f manifests/worker-proxy-service.yaml
daemonset.apps/boundary-worker created
service/worker-proxy created

# Verify all pods and services are up and running
$ kubectl get pods,svc
NAME                            READY   STATUS      RESTARTS   AGE
pod/boundary-controller-nn669   1/1     Running     0          67s
pod/boundary-worker-dzwqq       1/1     Running     0          56s
pod/database-init-58nnr         0/1     Completed   0          77s
pod/postgresql-0                1/1     Running     0          94s

NAME                         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)          AGE
service/controller-api       LoadBalancer   10.116.8.179   34.132.128.238   9200:30011/TCP   67s
service/controller-cluster   ClusterIP      10.116.4.164   <none>           9201/TCP         67s
service/postgresql           ClusterIP      10.116.7.183   <none>           5432/TCP         93s
service/worker-proxy         ClusterIP      None           <none>           9202/TCP         56s
```