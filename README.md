# Below are the steps and commands

#### Required Tools

- Java 8+
- Eclipse - Recent Version 
- Maven
- Git
- Docker
- Kubernetes
- ELB CLI
- Cloud Account



##### On client machine (Ubuntu) Install Azure CLI on Ubuntu Machine by participants
```
- curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

##### Connect to Kubernetes Cluster:
```
az login

az account list --refresh --output table

az account set --subscription "06cd1590-77ab-47ac-8ee2-79cabe2d5efb"

```

##### Install the Kubernetes CLI on Client Machines to connect to the Kubernetes Cluster
```
sudo az aks install-cli

#List and Connect to an AKS cluster using the Azure CLI
sudo az aks list -o table
sudo az aks get-credentials --resource-group agrg --name atingupta2005-cluster

#Using kubectl config get-contexts we'll be able to see all the clusters we've authenticated against, regardless what subscription they're in:
sudo kubectl config get-contexts

#Switch cluster:
sudo kubectl config use-context atingupta2005-cluster

#If there is permission issue and user need to use sudo then run below command:
sudo chown -R $USER ~/.kube

#Kubectl Autocomplete
source <(kubectl completion bash) # setup autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.

kubectl get namespaces
kubectl create namespace $USER
#kubectl delete namespaces $USER

#permanently save the namespace for all subsequent kubectl commands in that context.
kubectl config set-context --current --namespace=$USER

kubectl get nodes
```

### Commands Executed

```
# Run below commands to connect to Azure Kubernetes Cluster
# Use these commands if Azure Cluster and your namespace is already created
az login
sudo az aks get-credentials --resource-group agrg --name atingupta2005-cluster
# To install Kubectl refer: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows
sudo kubectl config use-context atingupta2005-cluster
source <(kubectl completion bash) # setup autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
kubectl get namespaces
kubectl config set-context --current --namespace=$USER
kubectl get nodes


#Login to ubuntu where Docker is installed and run:
sudo docker run -d -p 8080:8080 atingupta2005/hello-world-rest-api:0.0.1.RELEASE
docker ps
# Visit <publicIP>:8080/hello-world

#0.--------------To start Kubernetes Dashboard.
#Referenceses:
 - https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/aks/kubernetes-dashboard.md
 - https://blog.mobatek.net/post/ssh-tunnels-and-port-forwarding/

az aks browse --resource-group agrg --name atingupta2005-cluster

kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard

#Create a SSH Tunnel using below guide:
#https://blog.mobatek.net/post/ssh-tunnels-and-port-forwarding/
#Below are the values to be specified:
  My Computer->Forwared Port: 8001
  SSH Server: IP: <IP of Remote Machine>, Username: <Your User ID>, Port: 22
# Now connect and it would ask for password. Specifify correct password and continue.

#Now open the Dashboard using below URL:
 http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/

#1.-------Deploy Docker image in Kubernetes
kubectl create deployment hello-world-rest-api --image=atingupta2005/hello-world-rest-api:0.0.1.RELEASE
kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080
kubectl get service
kubectl get pods
kubectl scale deployment hello-world-rest-api --replicas=3
kubectl get pods

#2.--------Pods
#Pod is the smallest deployable unit. Can not have container without Pod. Container lives inside Pod
kubectl get pods
kubectl get pods -o wide

# To get details about the artifact
kubectl explain pods

# Get more details of the pod
kubectl describe pod hello-world-rest-api-58ff5dd898-9trh2	

kubectl get pods
kubectl delete pod hello-world-rest-api-58ff5dd898-62l9d
kubectl get pods

#--------Get list of various artifacts
kubectl get all
kubectl get events
kubectl get pods
kubectl get replicaset
kubectl get deployment
kubectl get service

#3.------Replicaset
kubectl get replicasets
# Since we are telling that how much replicas we need, a replicaset will take care of it.
kubectl scale deployment hello-world-rest-api --replicas=4
kubectl get pods
kubectl get replicaset
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get rs -o wide

# Lets delete one Pod and see how its auto created. Please note that use correct Pod ID
kubectl delete pod hello-world-rest-api-67c79fd44f-n6c7l
kubectl get pods -o wide
kubectl delete pod hello-world-rest-api-67c79fd44f-8bhdt
kubectl get pods -o wide

#4.------------Deployment
# Deployment ensures the release upgrades without any issue.
# We don’t want to have the downtime while release upgrade and Deployment plays a key role in achieving that
# There are many deployment strategies - Rolling Update
# Whenever we want to deploy new release of our existing application, we desire zero downtime
# If we change the image and that image is not existing, it will not deploy that
kubectl set image deployment hello-world-rest-api hello-world-rest-api=DUMMY_IMAGE:TEST

kubectl get service
# Now if we check in browser the old version is still running

#If we check our replicasets, then there are 2 replicasets. One with old image, other for error image
kubectl get rs -o wide

#We will see that there are 4 Pods, 3 are for old image, and new one has invalid status
kubectl get pods

#Check the details of the error pod
kubectl describe pod hello-world-rest-api-85995ddd5c-msjsm

#Check the series of events and notice in the last that how image has been marked as invalid
kubectl get events --sort-by=.metadata.creationTimestamp

#So there is rolling update is in force, it create 1 more replicaset and initiate 1 pod at a time and deletes 1 pod.
# Now let's set the correct image and see the effect
kubectl set image deployment hello-world-rest-api hello-world-rest-api=atingupta2005/hello-world-rest-api:0.0.2.RELEASE

#Now check in the browser, how the app version is changing gradually
kubectl get service

# Lets see the events which occurred. 
# You will notice that Deployment is regularly updating the replication configuration like:
# Old:2/New:1
# Old:1/New:2
# Old:0/New:3
kubectl get events --sort-by=.metadata.creationTimestamp

# Summary: Deployment is really useful to make sure that we are able to 
# update new releases of application without downtime
# Strategy used by Deployment is called "Rolling Update"

kubectl get pods -o wide
kubectl get rs

#5.---------------Services
#If we delete the Pod then its auto created. Also each time the Private IP of Pod is changed
kubectl delete pod hello-world-rest-api-67c79fd44f-n6c7l
kubectl get pods -o wide
kubectl delete pod hello-world-rest-api-67c79fd44f-8bhdt
kubectl get pods -o wide

#We don't want to give a new IP every time.
#We need a single IP. Kubernetes provides Service for that purpose.
#It does the mapping of Single IP with the POD IP
#Role of the service is to always provide the single IP

kubectl get service -o wide

#6.-----------Work with Eclipse Projects
#Till now we have used the prebuilt images
#Let's now create our own images and auto deploy them on Dockerhub
#Download 3 projects in Eclipse
 - 01-hello-world-rest-api
 - https://github.com/atingupta2005/01-hello-world-rest-api
 
 - 02-todo-web-application-h2
 - https://github.com/atingupta2005/02-todo-web-application-h2

 - 03-todo-web-application-mysql
 - https://github.com/atingupta2005/03-todo-web-application-mysql
 
#Now let’s setup the project # 1 in Eclipse
 - Make sure to Run As->As Maven Project

#Lets review the Dockerfile in the project

#Build the project Run As->Maven Project

#We will have the below flow:
# Project Code -> Push to GitHub -> Dockerhub senses the changes -> Docker Image is built automatically

#Now let's connect GitHub and Dockerhub

#Commit the project to GitHub and make sure that Jar files are also pushed

#Notice that the Docker Build is started on Docker hub and a new image is created on Dockerhub


#------------Auto scale
# To auto scale based on the workload of CPU
kubectl auto scale deployment hello-world-rest-api --max=10 --cpu-percent=70

kubectl set image deployment hello-world-rest-api hello-world-rest-api=atingupta2005/hello-world-rest-api:0.0.2.RELEASE

kubectl get pods

#7.---------- Deploy 01 Spring Boot Hello World Rest API to Kubernetes
#We will just change the image name in existing Deployment and the new release will be published
kubectl set image deployment hello-world-rest-api hello-world-rest-api=atingupta2005/hello-world-rest-api:0.0.4-SNAPSHOT

#Now refresh web page and see that new version is live

#Now let’s see the rollout history of our Deployment
kubectl rollout history deployment hello-world-rest-api

#To record which command was used to change deployment use --record
kubectl set image deployment hello-world-rest-api hello-world-rest-api=atingupta2005/hello-world-rest-api:0.0.4-SNAPSHOT --record

kubectl rollout history deployment hello-world-rest-api

#Check status of the rollout
kubectl rollout status deployment hello-world-rest-api

#We can also undo deployment
kubectl rollout undo deployment hello-world-rest-api --to-revision=3

kubectl rollout status deployment hello-world-rest-api

kubectl rollout undo deployment hello-world-rest-api --to-revision=3

kubectl rollout status deployment hello-world-rest-api

#Lets check the history again
kubectl rollout history deployment hello-world-rest-api

#Let's see the logs of the application
kubectl get pods
kubectl logs hello-world-rest-api-67c79fd44f-d6q9z

#Let's see the logs of the application and follow
kubectl logs hello-world-rest-api-67c79fd44f-d6q9z -f

#To continuously monitor the website every 2 seconds
# Get IP Address
kubectl get service
watch curl http://<ipaddress>:8080/hello-world
# Now duplicate the terminal to run commands on another console. 
# We can also tile the windows so that both will be shown together.


# Now see the logs and focus on the logs generated by curl

#8.-----------Generate Kubernetes YAML Configuration for Deployment and Service
#Till now we have used command line to create our pods and services
#In real world scenario we need to use YAML files

kubectl get deployment hello-world-rest-api

kubectl get deployment hello-world-rest-api -o wide
kubectl get deployment hello-world-rest-api -o yaml
kubectl get deployment hello-world-rest-api -o yaml > deployment.yaml
kubectl get service hello-world-rest-api -o yaml
kubectl get service hello-world-rest-api -o yaml > service.yaml

#9.-----------Understand and Improve Kubernetes YAML Configuration
#In Kubernetes we can define multiple resources in a single file
#We can cut all content from service.yaml and paste in deployment.yaml
#Also cleanup the final file and delete the unnecessary content
#Final cleaned up file:
  - https://raw.githubusercontent.com/atingupta2005/01-hello-world-rest-api/master/backup/deployment-01-after-cleanup.yaml

#Now let's delete the application and deploy using yaml file
#Here we are using the label to delete the resource
kubectl delete all -l app=hello-world-rest-api
kubectl get all
wget https://raw.githubusercontent.com/atingupta2005/01-hello-world-rest-api/master/backup/deployment-01-after-cleanup.yaml
vim deployment-01-after-cleanup.yaml
#----Imp: Make sure to change the ImagePullPolicy to Always
kubectl apply -f deployment-01-after-cleanup.yaml
kubectl get all

#Take the external IP and browse on the website at port 8080
watch kubectl get service

#10.----------------Understanding Kubernetes YAML Configuration - Labels and Selectors
#Open the final yaml file we created previously and understand it.
vim deployment-01-after-cleanup.yaml

#In any yaml file, the important parts are:
 - apiVersion
 - kind
 - metadata
 - spec

# -spec\template is the definition of the POD and details of the container.
  -- imagePullPolicy is to specify what would happen if image is modified or not present.
  -- Available imagePullPolicy options - Always and IfNotPresent
# To match the POD with Deployment labels and selectors are used.

#Now let's change the image tag in the Pod definition.
#- Do the changed in file: 
 - deployment-01-after-cleanup.yaml

vim deployment-01-after-cleanup.yaml

#We can check if there is any difference in deployed environment with our yaml file
kubectl diff -f deployment-01-after-cleanup.yaml

#If there are any changes we can deploy the configuration using yaml file
kubectl apply -f deployment-01-after-cleanup.yaml

#11.-----------------------Reduce release downtime with minReadySeconds
#When we update the image tag to update the release, there is a few seconds downtime
#To boot up a POD, it might take time and Kubernetes just assumes that since the new pod is now running, it deletes the old pod.
#New POD is running but it might take time to load the services inside it.
#Its better to tell Kubernetes to wait for some seconds before deleting the old POD

# To solve it specify below parameter inside spec: in yaml file -> spec.minReadySeconds above template tag:

minReadySeconds: 45

kubectl apply -f deployment-01-after-cleanup.yaml

# Notice that the downtime is now reduced


#12.-----------------Understanding Replica Sets in Depth - Using Kubernetes YAML Config
#Replicaset can work independent of Deployment
#If we only use Replicaset, then it would only focus on maintaining the required number replicas
#Replicaset do not work for revision upgrades if image name is changed
wget https://raw.githubusercontent.com/atingupta2005/01-hello-world-rest-api/master/backup/deployment-02-using-replica-set.yaml
#Now use command:
kubectl get all
kubectl delete all -l app=hello-world-rest-api
kubectl delete horizontalpodautoscaler.autoscaling/hello-world-rest-api

# It’s in fact Replicaset definition only. Check the kind property
vim deployment-02-using-replica-set.yaml

kubectl apply -f deployment-02-using-replica-set.yaml

watch -n 0.1 curl http://52.191.33.131:8080/hello-world

#We don't see the deployment
kubectl get all  

#If we change the image tag/name then below command will do no impact:
vim deployment-02-using-replica-set.yaml
kubectl get all
kubectl apply -f deployment-02-using-replica-set.yaml
kubectl get all   #We don't see the deployment

#We will not see any new pods as Replicaset only cares for maintaining number of replicas as needed
kubectl get pods 

#Now update the revision by deleting the pod:
kubectl delete pod <>

# New pod with new version if now created
kubectl get pods
kubectl get service
#Watch the curl output and notice that version is coming different as only 1 pod is updated with new image
watch http:<ip>:8080

kubectl delete pod <>  # Delete another older pod also
kubectl get pods    #Both the pods are now having latest release

#So it’s better to use deployment if we need automatic updates

#13.----------------Configure Multiple Kubernetes Deployments with One Service
#We can configure the connect a service to the pods using selectors.
#Each artifact in Kubernetes is assigned label/values.
#These label/values are used by other artifacts as selectors
kubectl delete all -l app=hello-world-rest-api


#Modify the file deployment-03-Multiple-Deployments-With-Same-Service.yaml to remove selector from service
#Below is the modified file
wget https://raw.githubusercontent.com/atingupta2005/01-hello-world-rest-api/master/backup/deployment-03-Multiple-Deployments-With-Same-Service.yaml

#Remove the label selector from the service to select multiple pods 
vim deployment-03-Multiple-Deployments-With-Same-Service.yaml
kubectl get all

kubectl apply -f deployment-03-Multiple-Deployments-With-Same-Service.yaml

# Get the External IP
kubectl get service
watch curl http://<externalIP>:8080/hello-world

# Notice that how the version of app is getting changed


#14.-------------Setting up 02 Spring Boot Todo Web Application in Local
#Import the project from 
 - https://github.com/atingupta2005/02-todo-web-application-h2.git

#Build the project using: Run As->Maven Build
 - In Goal Specify: clean install
 - Add the generated jar file in GitHub
 - Commit the changes
 - Push to GitHub

#The Docker image will be created automatically on Docker hub

#15.-------------Pushing Docker Image to Docker Hub for Spring Boot Todo Web App
# It's automatically being pushed in our case.

#16.-------------Using Kubernetes YAML Config to Deploy Spring Boot Todo Web App
kubectl delete all -l app=hello-world-rest-api
kubectl get pods
kubectl get all
rm -rf deployment.yaml
wget https://raw.githubusercontent.com/atingupta2005/02-todo-web-application-h2/master/backup/deployment.yaml

#Using ImagePullPolicy we can change our release on Kubernetes
#-----Make sure to change ImagePullPolicy to Always and image tag to latest
vim deployment.yaml
kubectl apply -f deployment.yaml
kubectl get all
#Wait for 10-20 seconds
kubectl get all
#Get the External IP and check on the browser - http://<ipaddress>:8080/login
#Username: atingupta2005, password: dummy

# Now if we do any changes in the code and commit to GitHub, Our Docker image will be built automatically.
# --- Important: Remember to build the jar file before pushing to GitHub
# Just run below command to get latest images on Kubernetes
kubectl rollout restart deployment

#Above command will pull the latest images and then the Pods will be recreated with the latest image.
#Refresh the browser and check the updates

#17.-------------Playing with Kubernetes Commands - Top Node and Pod
#Fetches PODS from all namespaces
kubectl get pods --all-namespaces

#Fetches PODS of specific label from current namespace
kubectl get pods -l app=todo-web-application-h2

#Fetches PODS of specific label from all namespaces
kubectl get pods -l app=todo-web-application-h2 --all-namespaces

kubectl get services --all-namespaces

kubectl get services --all-namespaces --sort-by=.spec.type

kubectl get services --all-namespaces --sort-by=.metadata.name

#To get details of the various things running in the cluster
kubectl cluster-info

#To get the utilization details of the nodes
kubectl top node

#To get the utilization details of the pods
kubectl top pod

#Short commands
kubectl get services
kubectl get svc
kubectl get ev  #Events 
kubectl get rs  # Replicaset

kubectl get ns  #Namespace
kubectl get nodes
kubectl get no  #Nodes
kubectl get po  #Pods

#18.---------------Code Review of 03 Java Todo Web Application MySQL
#Open the project in Eclipse
 - https://github.com/atingupta2005/03-todo-web-application-mysql

#19.---------------Running MySQL as Docker Container on Local
sudo docker run --detach --env MYSQL_ROOT_PASSWORD=dummypassword --env MYSQL_USER=todos-user --env MYSQL_PASSWORD=dummytodos --env MYSQL_DATABASE=todos --name mysql --publish 3306:3306 mysql:5.7


#20.--------------Create Docker Image for 03 Todo Web Application and Use Link to connect
# Now build our web application - https://github.com/atingupta2005/03-todo-web-application-mysql
# A Docker image will be created automatically in Dockerhub

# Now start a container of our application from the Docker Image
sudo docker run -d -p 8080:8080 --link=mysql --env RDS_HOSTNAME=mysql atingupta2005/todo-web-application-mysql

#21.----------Launching Containers in Custom Network
docker network ls
docker network create web-application-mysql-network
docker inspect web-application-mysql-network
docker run --detach --env MYSQL_ROOT_PASSWORD=dummypassword --env MYSQL_USER=todos-user --env MYSQL_PASSWORD=dummytodos --env MYSQL_DATABASE=todos --name mysql --publish 3306:3306 --network=web-application-mysql-network mysql:5.7
docker inspect web-application-mysql-network

#22.-----------Playing with Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

#Delete all the Docker containers
wget https://raw.githubusercontent.com/atingupta2005/03-todo-web-application-mysql/master/docker-compose.yml
# Now open the docker-compose.yml file and focus on the volume. We need a volume for data persistence
vim docker-compose.yml
docker-compose --version
docker-compose up

#23.--------------Using Kompose to generate Kubernetes Deployment Configuration
sudo curl -L https://github.com/kubernetes/kompose/releases/download/v1.21.0/kompose-linux-amd64 -o /usr/local/bin/kompose
sudo chmod a+x /usr/local/bin/kompose

#Make sure docker-compose.yml is in current directory
#Now run below command and it would generate few Kubernetes yaml files.
kompose convert

#24.---------------Review Kubernetes YAML for MySQL and Java Web Application
#Now let's review all the files generated one by one
vim mysql-database-data-volume-persistentvolumeclaim.yaml
vim mysql-deployment.yaml
vim mysql-service.yaml
vim todo-web-application-deployment.yaml
vim todo-web-application-service.yaml

#We can also download these files from the below URLs:
#It’s better to use the cleaned up files from below URLs.
rm -rf mysql-database-data-volume-persistentvolumeclaim.yaml mysql-deployment.yaml mysql-service.yaml todo-web-application-deployment.yaml todo-web-application-service.yaml
wget https://raw.githubusercontent.com/atingupta2005/03-todo-web-application-mysql/master/backup/01-updated-manually/mysql-database-data-volume-persistentvolumeclaim.yaml
wget https://raw.githubusercontent.com/atingupta2005/03-todo-web-application-mysql/master/backup/01-updated-manually/mysql-deployment.yaml
wget https://raw.githubusercontent.com/atingupta2005/03-todo-web-application-mysql/master/backup/01-updated-manually/mysql-service.yaml
wget https://raw.githubusercontent.com/atingupta2005/03-todo-web-application-mysql/master/backup/01-updated-manually/todo-web-application-deployment.yaml
wget https://raw.githubusercontent.com/atingupta2005/03-todo-web-application-mysql/master/backup/01-updated-manually/todo-web-application-service.yaml

#25.---------------Deploy MySQL Database to Kubernetes Cluster
kubectl delete all -l app=todo-web-application-h2
kubectl apply -f mysql-database-data-volume-persistentvolumeclaim.yaml,mysql-deployment.yaml,mysql-service.yaml
kubectl get svc
kubectl get all
kubectl get pods

#Connect to mysql
#Change service to load balancer to get External IP - mysql-service.yaml
sudo snap install mysql-shell
mysqlsh
\connect todos-user@52.150.39.158
#password is dummytodos
\sql
use todos;
select * from todos;
## Todo table is created when we launch the app
\q

#-------Note 2-3 minutes to allow the containers of mysql spin-up
kubectl apply -f todo-web-application-deployment.yaml,todo-web-application-service.yaml
kubectl get all
kubectl get pods
kubectl get service

kubectl logs todo-web-application-b65cc44d9-7h9pr -f

kubectl get service

# Now browse the app and. Username: in28minutes or atingupta2005. Password: dummy

#-----------Next let's stop our config values and passwords centrally in persistent volume
rm -rf mysql-database-data-volume-persistentvolumeclaim.yaml mysql-deployment.yaml mysql-service.yaml todo-web-application-deployment.yaml todo-web-application-service.yaml
wget https://raw.githubusercontent.com/atingupta2005/03-todo-web-application-mysql/master/backup/02-final-backup-at-end-of-course/mysql-database-data-volume-persistentvolumeclaim.yaml
wget https://raw.githubusercontent.com/atingupta2005/03-todo-web-application-mysql/master/backup/02-final-backup-at-end-of-course/mysql-deployment.yaml
wget https://raw.githubusercontent.com/atingupta2005/03-todo-web-application-mysql/master/backup/02-final-backup-at-end-of-course/mysql-service.yaml
wget https://raw.githubusercontent.com/atingupta2005/03-todo-web-application-mysql/master/backup/02-final-backup-at-end-of-course/todo-web-application-deployment.yaml
wget https://raw.githubusercontent.com/atingupta2005/03-todo-web-application-mysql/master/backup/02-final-backup-at-end-of-course/todo-web-application-service.yaml

kubectl get pv
kubectl get pvc
kubectl describe pod/mysql-5ccbbbdcd8-5zjqg 

kubectl create configmap todo-web-application-config --from-literal=RDS_DB_NAME=todos
kubectl get configmap todo-web-application-config
kubectl describe configmap/todo-web-application-config

kubectl edit configmap/todo-web-application-config
  RDS_DB_NAME: todos
  RDS_HOSTNAME: mysql
  RDS_PASSWORD: dummytodos
  RDS_PORT: "3306"
  RDS_USERNAME: todos-user

kubectl apply -f todo-web-application-deployment.yaml 
kubectl scale deployment todo-web-application --replicas=0
kubectl scale deployment todo-web-application --replicas=1

kubectl create secret generic todo-web-application-secrets --from-literal=RDS_PASSWORD=dummytodos
kubectl get secret/todo-web-application-secrets
kubectl describe secret/todo-web-application-secrets

kubectl apply -f todo-web-application-deployment.yaml 
kubectl scale deployment todo-web-application --replicas=0
kubectl scale deployment todo-web-application --replicas=1

#kubectl delete -f mysql-database-data-volume-persistentvolumeclaim.yaml,mysql-deployment.yaml,mysql-service.yaml,todo-web-application-deployment.yaml,todo-web-application-service.yaml

#25-------------------Deploying Basic Spring Boot Microservices to Kubernetes
#Open below projects in Eclipse:
 - https://github.com/atingupta2005/04-currency-exchange-microservice-basic
 - https://github.com/atingupta2005/05-currency-conversion-microservice-basic

#Build both the projects and upload to github along with jar files

#Push the built project to GITHub

#Download Kubernetes YAML Files for below URLs:
rm -rf deployment_currency_exchange.yaml deployment_currency_conversion.yaml 00-configmap-currency-conversion.yaml ingress.yaml ingress_azure.yaml
wget https://raw.githubusercontent.com/atingupta2005/04-currency-exchange-microservice-basic/master/00-configmap-currency-conversion.yaml
wget -O  deployment_currency_exchange.yaml  https://raw.githubusercontent.com/atingupta2005/04-currency-exchange-microservice-basic/master/deployment.yaml
wget -O deployment_currency_conversion.yaml https://raw.githubusercontent.com/atingupta2005/05-currency-conversion-microservice-basic/master/deployment.yaml
wget https://raw.githubusercontent.com/atingupta2005/05-currency-conversion-microservice-basic/master/ingress.yaml
wget https://raw.githubusercontent.com/atingupta2005/05-currency-conversion-microservice-basic/master/ingress_azure.yaml

kubectl apply -f deployment_currency_exchange.yaml
kubectl get all
kubectl get service

curl http://52.170.172.91:8000/currency-exchange/from/USD/to/INR

kubectl apply -f deployment_currency_conversion.yaml
kubectl get all
kubectl get service

http://<>:8100/currency-conversion/from/EUR/to/INR/quantity/10

## Question: How services discover each other?
 - Refer to proxy java file -> CURRENCY_EXCHANGE_SERVICE_HOST
 - This Environment Variable is auto created by following conversion and the IP is stored in it
 - Verify by:
    - kubectl log <podname> -f 
 - The approach above can make a problem. Since the above environment variable is created only after service is created.
 - What will happen if the service is not created before the other service which is dependent.
 - To solve, we can also make use of CURRENCY_EXCHANGE_URI: http://currency-exchange
   - Refer to deployment file and proxy

kubectl logs currency-conversion-76fd9b6958-tt7vk | grep CURRENCY_EXCHANGE_URI
kubectl logs currency-conversion-76fd9b6958-tt7vk | grep CURRENCY_EXCHANGE
kubectl logs currency-conversion-76fd9b6958-tt7vk | grep CURRENCY_EXCHANGE_SERVICE_HOST

kubectl create configmap currency-conversion --from-literal=YOUR_PROPERTY=value --from-literal=YOUR_PROPERTY_2=value2

kubectl delete deployment.apps/currency-conversion

kubectl apply -f deployment_currency_conversion.yaml

#-------Ingress
vim ingress_azure.yaml

#Review the yaml file

kubectl apply -f ingress_azure.yaml

#Ingress creation take approx 10 minutes. So we need to wait

# By the time lets review how the LoadBalancer is connected with Azure Portal

#Now let's see the ingress details. If it's still not showing IP address then wait for another 5-10 minutes. 
kubectl get ingress

#Now since we don't need Load Balancers for our Microservices, we will change type from LoadBalancer to NodePort
vim deployment_currency_conversion.yaml
#Change type of service to NodePort

vim deployment_currency_exchange.yaml
#Change type of service to NodePort


#Check that the services are of type LoadBalancers
kubectl get svc

kubectl apply -f deployment_currency_exchange.yaml
kubectl apply -f deployment_currency_conversion.yaml
 
#Now check if Load Balancers have been changed to NodePort
kubectl get svc

Once we get the IP of Ingress, check if you are able to open both the Microservices from the same IP.
curl http://<IngressIP>/currency-exchange/from/USD/to/INR
curl http://<IngressIP>/currency-conversion/from/EUR/to/INR/quantity/10


#----------Auto Scaling
#Horizontal
#Vertical
#Can scale as per:
 - Cluster Autoscaling: Number of Nodes. While creating the Cluster enable the autoscaling features
 - Horizontal POD Autoscaling: Number of Pods
 - Vertical POD Autoscaling: Increase resources(RAM?CPU) of the POD. Need to specify while creating the cluster

#Horizontal POD Autoscaling 
wget -O deploment_currency_exchange_autoscale.yaml https://raw.githubusercontent.com/atingupta2005/04-currency-exchange-microservice-basic/master/deployment.yaml
vim deploment_currency_exchange_autoscale.yaml

#Review the file for autoscaling features
Modify the CPU usage Limists. Min 100, max 500
kubectl deployment -f deploment_currency_exchange_autoscale.yaml
kubectl get pods
kubectl logs <podID> -f
kubectl top pods
watch -n 0.1 curl curl http://<IngressIP>/currency-conversion/from/EUR/to/INR/quantity/10
watch -n 0.1 kubectl top pods

#Let's autoscale deployment based on CPU usage
kubectl auto scale deployment currency-exchange --min=1 --max=3 --cpu-percent=10 
watch -n 0.1 kubectl get pods  # PODS will be created/deleted automatically based on the Load

#Let' see the events.
kubectl get events

#Use yaml style for the same
kubectl get hpa
kubectl get hpa -o yaml
kubectl get hpa currency-exchange -o yaml > 01-hpa.yaml

vim hpa.yaml

#Review the content and delete metadata

#Helm Charts--------------------------------------------------------
#Installing Help Client:
#Refer - https://helm.sh/docs/intro/install/
#For Linux:
 sudo snap install helm --classic
 
#Installing Tiller
wget https://raw.githubusercontent.com/atingupta2005/kubernetes-java-projects/master/12-helm/helm-tiller.sh
vim helm-tiller.sh 
sh helm-tiller.sh
helm init
helm init --upgrade
helm init --service-account tiller --upgrade


#Practical Examples
wget https://raw.githubusercontent.com/atingupta2005/kubernetes-java-projects/master/12-helm/currency-conversion.zip
wget https://raw.githubusercontent.com/atingupta2005/12-helm/master/currency-exchange.zip
sudo apt install -y  unzip
unzip currency-conversion.zip
unzip currency-exchange.zip

cd  currency-exchange

vim Chart.yaml
vim values.yaml
cd templates

#Variables are used in this template
microservice-deployment.yaml

#Install Helm Chart
helm install ./currency-exchange/ --name=currency-services


#Install Helm Chart
helm install ./currency-conversion/ --name=currency-services-1


#Before Install Helm Chart, do a dry run
helm install ./currency-conversion/ --name=currency-services-3 --debug --dry-run

#Get the IP Address:
kubectl get all
kubectl get svc
kubectl get pods

#Visit:
curl <IP>:8000/currency-exchange/from/EUR/to/INR
curl <IP>:8100/currency-exchange/from/EUR/to/INR/quantity/10

#To see a history of Helm Chart Deployment
helm history currency-services-1

#Do some changes in yaml file
vim values.yaml

#Upgrade the help chart to reflect the changes
helm upgrade currency-services-1 ./currency-conversion/

#We can also rollback
helm history currency-services-1
helm rollback currency-services-1 1

helm history currency-services-1

kubectl delete all -l app=currency-exchange

helm repo list

helm search

helm search mysql

helm inspect stable/mysql

helm inspect values stable/mysql

#----------------------------------------------------

#Extra commands:
#Generally when the status of your pod is ImgaePullBackOff the sytem automatically tries to 
#re-pull the image in a matter of minutes, 
#if you actually want to restart the pod then 
#you'll probably have to delete the pod and recreate it.
kubectl replace --force -f <pod yaml file>

kubectl delete --all namespaces
kubectl delete --all deployments --namespace=foo
kubectl delete pods --all
kubectl delete deployment --all
kubectl delete rc --all
kubectl delete rs --all
kubectl delete service -l provider=fabric8
kubectl delete secret -l provider=fabric8
kubectl delete ingress --all
kubectl delete configmap --all
kubectl delete sa --all


#Restart Deployment
kubectl rollout restart <deployment-name>
#We can just scale down and scale up to redeploy
kubectl scale deployment todo-web-application --replicas=0
kubectl scale deployment todo-web-application --replicas=1


#Point to note/remember:
 - If we create deployment without YAML file then latest image is not pulled. 
 - We need to use a new image tag to force a image repull


#Extra leraning topics:
# How to assign the POD to specific Node.
https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes/

#Referenes:
Cheatcheat:
https://kubernetes.io/docs/reference/kubectl/cheatsheet/

Kubernetes Tools
https://dzone.com/articles/50-useful-kubernetes-tools
https://techbeacon.com/enterprise-it/9-top-open-source-tools-monitoring-kubernetes
https://logz.io/blog/open-source-monitoring-tools-for-kubernetes/

```

## Azure

```
az group create --name kubernetes-resource-group --location westeurope
az ad sp create-for-rbac --skip-assignment --name kubernetes-cluster-service-principal
az aks create --name atingupta2005-cluster --node-count 4 --enable-addons monitoring --resource-group kubernetes-resource-group --vm-set-type VirtualMachineScaleSets --load-balancer-sku standard --enable-cluster-autoscaler  --min-count 1 --max-count 7 --generate-ssh-keys --service-principal HIDDEN --client-secret  HIDDEN
az aks get-credentials --resource-group kubernetes-resource-group --name atingupta2005-cluster
kubectl version
kubectl get nodes
kubectl create deployment hello-world-rest-api --image=atingupta2005/hello-world-rest-api:0.0.1.RELEASE
kubectl expose deployment hello-world-rest-api --type=LoadBalancer --port=8080
kubectl create deployment todowebapp-h2 --image=atingupta2005/todo-web-application-h2:0.0.1-SNAPSHOT
kubectl expose deployment todowebapp-h2 --type=LoadBalancer --port=8080
git clone https://github.com/atingupta2005/kubernetes-java-projects.git
cd 03-todo-web-application-mysql/backup/02-final-backup-at-end
kubectl apply -f mysql-database-data-volume-persistentvolumeclaim.yaml,mysql-deployment.yaml,mysql-service.yaml
kubectl apply -f config-map.yaml,secret.yaml,todo-web-application-deployment.yaml,todo-web-application-service.yaml
kubectl delete all -l app=hello-world-rest-api
kubectl delete all -l app=todowebapp-h2
kubectl delete all -l io.kompose.service=todo-web-application
kubectl delete all -l io.kompose.service=mysql

cd ../../..
kubectl apply -f 04-currency-exchange-microservice-basic/deployment.yaml
kubectl apply -f 05-currency-conversion-microservice-basic/deployment.yaml

helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm install stable/nginx-ingress --namespace default --set controller.replicaCount=1 --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux --generate-name

kubectl apply -f 05-currency-conversion-microservice-basic/ingress_azure.yaml

kubectl get svc
kubectl delete svc currency-conversion
kubectl delete svc currency-exchange
kubectl delete svc nginx-ingress-1583140351-controller

az aks delete --name atingupta2005-cluster --resource-group kubernetes-resource-group

```
