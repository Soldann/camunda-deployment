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
On the first install, run the following command:
```
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```
Now deploy the MetalLB load balancer.
```
kubectl apply -f metallb-configMap.yaml
kubectl apply -f metallb-deploy.yaml
```
### Deploy nginx ingress
```
kubectl apply -f nginx-deploy.yaml
```
This deploys nginx-ingress

### Deploy Postgresql and the operator
```
kubectl apply -f postgres-operator-configMap.yaml
kubectl apply -f postgres-pod-config.yaml
kubectl apply -f postgres-operator/manifests/operator-service-account-rbac.yaml
kubectl apply -f postgres-operator/manifests/postgres-operator.yaml
```
Wait for the postgres-operator pod to finish deploying, then run the following to create the databases
```yaml
kubectl apply -f database-setup.yaml
```

### Deploy Camunda
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
