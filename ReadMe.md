# CoreOps AKS Kubernetes Hands-on Workshop

### Prerequisites

1. Docker (https://docs.docker.com/get-docker/)
2. kubectl (https://kubernetes.io/docs/tasks/tools/)
3. kubelogin (https://azure.github.io/kubelogin/install.html)
4. copsctl (https://github.com/conplementAG/copsctl#installation)
5. helm (https://helm.sh/docs/intro/quickstart/#install-helm)

## Workshop

### 0. Kubernetes basics

Explain the concept of nodes & containers, pods, namespaces, resources. Mention deployments & services, but explain them hands on in next sections.


### 1. Connect to the cluster

- get the connection string from the CoreOps team

```bash
copsctl connect -e <env> -c <connection_string>
```
- connect, login
- try to execute `kubectl get pods` - you will get an error (as expected) - note the username shown here! You will need this in the next step. 

### 2. Create own namespace

```bash
copsctl namespace create -n ws-xxx --users <your_username_here> -c 31523 -p cp-workshop
```

### 3. Deploy the Redis backend (Kubernetes Deployment)

```bash
cd 01_backend_redis

kubectl apply -f deployment.yaml -n ws-xxx

kubectl get deployment -n ws-xxx
kubectl get pod -n ws-xxx

kubectl describe pod <pod-name> -n ws-xxx
```

### 4. Test the Redis backend (Kubernetes Service)

```bash
kubectl exec --stdin --tty <pod-name> --namespace ws-xxx -- /bin/bash

kubectl apply -f service.yaml --namespace ws-xxx

kubectl get deployment,service --namespace ws-xxx

kubectl get pod -n ws-xxx

kubectl exec --stdin --tty <pod-name> --namespace ws-xxx -- /bin/bash

redis-cli
SET KEY VALUE
GET KEY
```

# create a test container with redis-cli, so that we can connect
# to redis and see if the container works
```bash
kubectl run redis-cli --rm -it -n ws-xxx --image redis -- /bin/bash

kubectl get pod -n ws-xxx -o wide

redis-cli -h <pod-cluster-ip> PING

kubectl get service -n ws-xxx
redis-cli -h <service-cluster-ip> PING

cat /etc/resolv.conf

redis-cli -h azure-vote-back PING
redis-cli -h azure-vote-back.ws-xxx.svc.cluster.local PING


redis-cli -h azure-vote-back

# now you can play around with set x value; get x; etc...
SET KEY VALUE
GET KEY
```

### 5. Create the frontend (Kubernetes Deployment & Service)

```bash
cd ..
cd 02_frontend_dotnet

kubectl apply -f deployment-and-service.yaml --namespace ws-xxx
kubectl get deploy,service --namespace ws-xxx
```

### 6. Test the frontend

- test via curl inside the cluster

```bash
kubectl run curl-cli --rm -it -n ws-xxx --image=radial/busyboxplus:curl

# then inside the container:
curl azure-vote-front
nslookup azure-vote-front
```

- test via port-forwarding to the local machine

```bash
kubectl port-forward --namespace ws-xxx svc/azure-vote-front 8080:80

# browser -> localhost:8080
```

### 7. Expose the frontend to the public Internet

```bash
cd ..
cd 03_ingress

kubectl apply -f ingress.yaml --namespace ws-xxx

vote.xxx.cpone.conplement.cloud

kubectl get ingress --namespace ws-xxx

# to check the certificate you can also execute:
kubectl get certificates --namespace ws-xxx
kubectl get certificaterequests --namespace ws-xxx

nslookup vote.xxx.cpone.conplement.cloud
```

### 8. Load testing & autoscaling

```bash
kubectl scale --replicas 3 deployment/azure-vote-front -n ws-xxx

cd ..
cd 04_load_test

# run the load test before deploying autoscaling, observe
# powershell
Get-Content script.js | docker run -i loadimpact/k6 run --vus 10 --duration 60s --insecure-skip-tls-verify -

# shell
docker run -i loadimpact/k6 run --vus 10 --duration 60s  --insecure-skip-tls-verify - <script.js

# with local k6
k6 run .\script.js --vus 10 --duration 60s --insecure-skip-tls-verify

# deploy the autoscaling functionality, now you can run the load test again and observe
kubectl apply -f hpa.yaml -n ws-xxx

kubectl get horizontalpodautoscaler -n ws-xxx # or 'kubectl get hpa' for short
```

### 9. Packaging and interpolation via Helm

```bash
cd ..
cd 05_helm

helm create azure-vote
```

Now, you can delete these sections:
- charts
- values.yaml file containts (leave the empty file)
- everything inside templates folder

Copy your resources into your helm chart. Interpolate some values / variables. Release the chart.

```bash
helm upgrade --wait --timeout 15m --install --namespace ws-xxx azure-vote .
[--take-ownership]

kubectl delete deployment -n ws-xxx azure-vote-front azure-vote-back
kubectl delete service -n ws-xxx azure-vote-front azure-vote-back
kubectl delete ingress -n ws-xxx azure-vote-front-ingress-coreops-public

helm uninstall azure-vote -n ws-xxx
copsctl namespace delete -n ws-xxx
```

### 10. Discussion around key concepts behind CoreOps and DevOps split

- where are the application Azure resources located?
- who operates these resources (SQL DB etc.)?
- how many subscriptions are there?
