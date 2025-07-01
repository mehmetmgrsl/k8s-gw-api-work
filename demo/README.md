### 1- Prerequisites

- You already have:
   - **minikube** (host cluster) -> [CLUSTER.md](./CLUSTER.md)
   - **kubectl**

### 2- Install Gateway API Resources

   - **The standard release channel** includes all resources that have graduated to GA or beta, including GatewayClass, Gateway, HTTPRoute, and ReferenceGrant. To install this channel, run the following kubectl command:

```
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.3.0/standard-install.yaml


# Optional: verify
kubectl get crd | grep gateway
```

### 3 - Configure NGINX Gateway Fabric

- NGINX Gateway Fabric, which is a controller that watches Gateway API resources and configures NGINX accordingly.

```
# Deploy NGINX Gateway Fabric CRDs
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/crds.yaml

# Deploy NGINX Gateway Fabric
# This creates a Deployment and Service in the nginx-gateway namespace
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v1.6.1/deploy/nodeport/deploy.yaml

# Verify the deployment
kubectl get pods -n nginx-gateway
```

### 4 - Let's configure specific ports for the gateway service:

```
# View the nginx-gateway service
kubectl get svc -n nginx-gateway nginx-gateway -o yaml

# Update the service to expose specific nodePort values
kubectl patch svc nginx-gateway -n nginx-gateway --type='json' -p='[
  {"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30080},
  {"op": "replace", "path": "/spec/ports/1/nodePort", "value": 30081}
]'

# Optional: Verify the service has been updated
kubectl get svc -n nginx-gateway nginx-gateway
```

- This exposes the Gateway on your local machine using ports 30080 (HTTP) and 30081 (HTTPS), which will be used in testing.


### 5 - Create GatewayClass and Gateway Resources

```
kubectl apply -f gateway-resources.yaml

# Optional: Verify resources
kubectl get gatewayclass
kubectl get gateway -n nginx-gateway
kubectl describe gateway nginx-gateway -n nginx-gateway
```

- GatewayClass: describes the kind of Gateway you're using (NGINX here).
- Gateway: defines how/where traffic enters (port, protocol, allowed routes).
- Think of this like setting up the entry point to your cluster.




### 6 - Create a Pod named frontend-app with container port 80

```kubectl apply -f pod.yaml```

### 7 - Create a Service to expose it on port 80, forwarding to 80

```kubectl apply -f service.yaml```

### 8 - Expose the Frontend Service

- This defines the routing rule for incoming traffic.
- All traffic with path / will go to frontend-svc on port 80.
- HTTPRoute tells the Gateway what traffic to forward where.

```
kubectl apply -f frontend-route.yaml

# Verify the HTTPRoute
kubectl get httproute frontend-route -n nginx-gateway
kubectl describe httproute frontend-route -n nginx-gateway
```

You should see the HTTPRoute created and showing as "Accepted".

### 9 - Test Internally from Minikube (Recommended for macOS/Docker users)


```
minikube ssh
curl http://localhost:30080/

```

### 10 - Clean-up after Demo

- This removes all resources and namespaces created during the demo to leave your cluster clean.

```
kubectl delete -f frontend-route.yaml
kubectl delete -f service.yaml
kubectl delete -f pod.yaml
kubectl delete -f gateway-resources.yaml
kubectl delete ns nginx-gateway
```

### References
(1*) [Kubernetes Gateway API - Tech Tutorials with Piyush](https://www.youtube.com/watch?v=2lcx74HUz1U)

(2*) https://gateway-api.sigs.k8s.io/guides/

