# camunda-deployment
Kubernetes deployment yaml file

nginx deployment modified from https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/deploy.yaml

MetalLB version 0.9.3, merged from https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml, https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml

## How to deploy

### If installing on kind:
```
kind create cluster --config config.yaml
```
This creates a cluster with one master and two worker nodes

### Deploy MetalLB
The following deploys the MetalLB load balancer.
```
kubectl apply -f metallb-deploy.yaml
kubectl apply -f metallb-configMap.yaml
```
On the first install, run the following command:
```
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```
### Deploy nginx ingress
```
kubectl apply -f nginx-deploy.yaml
```
This deploys nginx-ingress

### Deploy Postgresql and the operator
```
kubectl apply -f postgres-operator-configmap.yaml
kubectl apply -f postgres-pod-config.yaml
kubectl apply -f postgres-operator/manifests/operator-service-account-rbac.yaml
kubectl apply -f postgres-operator/manifests/postgres-operator.yaml
```
Wait for the postgres-operator pod to finish deploying, then run the following to create the databases
```yaml
kubectl apply -f database-setup.yaml
```

### Deploy Camunda
If Kubernetes does not have a default storageclass with provisioner configured, either apply the manual persistent volumes:
```
kubectl apply -f persistentVolumes.yaml
```
OR use a provisioner like https://github.com/rancher/local-path-provisioner
```
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

Wait for the database pods to finish deploying before starting applying Camunda.
```
kubectl apply -f deployment.yaml
```

## Debugging
You can debug with the included alpine pod:
```
kubectl apply -f alpine.yaml
kubectl exec -it alpine -- sh #enter pod
apk add postgresql-client #download psql client
export PGPASSWORD=$DB_PASSWORD #set password for psql
psql -h camunda-db.default.svc.cluster.local -U camunda -d process_engine
```
