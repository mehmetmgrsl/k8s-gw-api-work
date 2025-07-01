- The script below 
  - starts a Minikube cluster named host-k8s-cluster using Docker, 
  - sets it as the active profile, 
  - enables the Ingress addon, 
  - waits for the NGINX ingress controller to become ready, 
  - and finally exports the Minikube IP to the INGRESS_HOST environment variable for use in routing traffic.

```
minikube start -p host-k8s-cluster --driver=docker --bootstrapper=kubeadm --kubernetes-version=latest --force

minikube profile host-k8s-cluster

minikube addons enable ingress


kubectl --namespace ingress-nginx wait \
    --for=condition=ready pod \
    --selector=app.kubernetes.io/component=controller \
    --timeout=120s


export INGRESS_HOST=$(minikube ip)
echo "INGRESS_HOST:"
echo $INGRESS_HOST
```