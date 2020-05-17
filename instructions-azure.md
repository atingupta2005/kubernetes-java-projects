# Kubernetes on Azure

## Azure

### Create AKS Cluster

Let's use Azure Cloud Shell

```
# az login # az cli

az group create --name kubernetes-resource-group --location westeurope

az ad sp create-for-rbac --skip-assignment --name kubernetes-cluster-service-principal

az aks create --name atingupta2005-cluster --node-count 4 --enable-addons monitoring --resource-group kubernetes-resource-group --vm-set-type VirtualMachineScaleSets --load-balancer-sku standard --enable-cluster-autoscaler  --min-count 1 --max-count 7  --generate-ssh-keys --service-principal <<appId>> --client-secret  <<password>>

az aks get-credentials --resource-group kubernetes-resource-group --name atingupta2005-cluster
```

### Deploying Basic Applications

```
kubectl create deployment hello-world-rest-api --image=atingupta2005/hello-world-rest-api:0.0.1.RELEASE

kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080

kubectl create deployment todowebapp-h2 --image=atingupta2005/todo-web-application-h2:0.0.1-SNAPSHOT

kubectl expose deployment todowebapp-h2 --type=LoadBalancer --port=8080

cd kubernetes-java-projects/03-todo-web-application-mysql/backup/02-final-backup-at-end 
kubectl apply -f mysql-database-data-volume-persistentvolumeclaim.yaml,mysql-deployment.yaml,mysql-service.yaml

#WAIT FOR MYSQL TO BE UP AND RUNNING

kubectl apply -f config-map.yaml,secret.yaml,todo-web-application-deployment.yaml,todo-web-application-service.yaml
```

### Delete Basic Applications

```
kubectl delete all -l app=hello-world-rest-api
kubectl delete all -l app=todowebapp-h2
kubectl delete all -l io.kompose.service=todo-web-application
kubectl delete all -l io.kompose.service=mysql
```

### Deploy Microservices

```
kubectl apply -f 04-currency-exchange-microservice-basic/deployment.yaml 
kubectl apply -f 05-currency-conversion-microservice-basic/deployment.yaml
# watch -n 0.1 curl
# Explore Logs
```

### Create Ingress

Install Ingress Controller
```
helm repo add stable https://kubernetes-charts.storage.googleapis.com/

helm install stable/nginx-ingress --namespace default --set controller.replicaCount=1 --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux --generate-name

kubectl get service -l app=nginx-ingress
```

More details - https://docs.microsoft.com/en-us/azure/aks/ingress-tls

Setup Ingress
```
kubectl apply -f 05-currency-conversion-microservice-basic/ingress_azure.yaml
```
### Exploring Cluster Auto Scaler

Testing auto scaler
```
kubectl create deployment autoscaler-demo --image=nginx
kubectl scale deployment autoscaler-demo --replicas=5000
```

### Clean up the cluster

Delete all LoadBalancer services
```
kubectl get svc --all-namespaces
kubectl delete svc service-name
```

Delete the cluster
```
az aks delete --name atingupta2005-cluster --resource-group kubernetes-resource-group
```