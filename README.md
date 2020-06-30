# camunda-deployment
Kubernetes deployment yaml file

nginx deployment modified from https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/deploy.yaml

MetalLB version 0.9.3, merged from https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml, https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml

Run this command on first install only:
- kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
