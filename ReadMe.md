# CoreOps AKS Kuberentes Hands-on Workshop

### Prerequisites

1. Docker (https://docs.docker.com/get-docker/)
2. kubectl (https://kubernetes.io/docs/tasks/tools/)
3. copsctl (https://github.com/conplementAG/copsctl#installation)
4. helm (https://helm.sh/docs/intro/quickstart/#install-helm)

## Workshop

### 0. Kubernetes basics

Explain the concept of nodes & containers, pods, namespaces, resources. Mention deployments & services, but explain them hands on in next sections.


### 1. Connect to the cluster

- get the connection string from the CoreOps team
- connect, login
- try to execute `kubectl get pods` - you will get an error (as expected) - note the username shown here! You will need this in the next step. 

### 2. Create own namespace

```bash
copsctl namespace create -n ws-xxx --users your_username_here
```

### 3. Deploy the Redis backend (Kubernetes Deployment)

```bash
cd 01_backend_redis

kubectl apply -f deployment.yaml --namespace ws-xxx
kubectl get deployment -n ws-xxx
```

### 4. Test the Redis backend (Kubernetes Service)

```bash
kubectl apply -f service.yaml --namespace ws-xxx
kubectl get deploy,service --namespace ws-xxx

# create a test container with redis-cli, so that we can connect
# to redis and see if the container works
kubectl run redis-debug-pod-with-cli --rm -it --namespace ws-xxx --image redis -- /bin/bash
# now you can play around with set x value; get x; etc...
```

### 5. Create the .NET frontend (Kubernetes Deployment & Service)

```bash
cd ..
cd 02_frontend_dotnet

kubectl apply -f deployment-and-service.yaml --namespace ws-xxx
kubectl get deploy,service --namespace ws-xxx
```

### 6. Test the .NET frontend

- test via curl inside the cluster

```bash
kubectl run curl-cli -n ws-xxx --image=radial/busyboxplus:curl -i --tty --rm

# then inside the container:
curl azure-vote-front
```

- test via port-forwarding to the local machine

```bash
kubectl port-forward --namespace ws-xxx svc/azure-vote-front 8080:80

# browser -> localhost:8080
```

### 7. Expose the .NET frontend to the public Internet

```bash
cd ..
cd 03_ingress

kubectl apply -f ingress.yaml --namespace ws-xxx

kubectl get ingress --namespace ws-xxx

# to check the certificate you can also execute:
kubectl get certificates --namespace ws-xxx
```