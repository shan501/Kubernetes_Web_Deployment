# Kubernetes Web Deployment
In this guide i will go over how to use Kubernetes to deploy a web app with a mangodb database attached.I will use Minikube to set 
up a local cluster.I will create a config , service , and deployment file using yaml.I used Kubernetes documentation for this project as well
as this youtube video https://www.youtube.com/watch?v=s_o8dwzRlu4&t=1138s.

## Introduction
ConfigMap and Secrets are external configuration.These are for storing configuration details that you dont want to hard code in the deployment file.
When you hard code configurations in the deployment file and you want to change it , you would need to build everything from scratch again.You would need
to build a new image, push it to repo , pull it in your pod , and deploy your pot all over again.The Secrets are used to store confidentiaal information
such as username and password to a database server where configmap stores insensitive information such as the mapping to a Service.


## Installation 
Minikube can run either on a virtual machine or as a docker container.I will be using windows to install Minikube.First you would have to 
download Docker Desktop for Windows.Then you need to download Minikube either by downloading the exe file or Powershell.Then run the command
minikube start. Minikube CLI is for starting and deleteing the cluster and you use kubectl to interact with the cluster.


## Create a config map
I will create a configMap to map the database to the web-app.The web-app will use the key under mango-url to know what database
to map to.
```
apiVersion: v1
kind: ConfigMap
metadata:
  name : mango-config-map
data:
  mongo-url : mango-service 
```
The name of this is called mango-config-map .The data attribute maps to the mango database that we will create later.We will attach a service
to the mangodb database.The mango-url maps the mango-service that we are going to set up latter.The service gives the database an IP that will not
be changed on reboot

## Create a Secret
We will create a secret to hold the username and password to our database
```
apiVersion: v1
kind:secret
metadata:
  name:mango-secret
type: Opaque
data:
  mango-user: "encoded base64 password"
  mango-password:"encoded base64 password"

```
The type Opaque is the default value for secets.The kind is a scret, and the data contains the username and password.It needs to be 
encoded in base64.

## Create configuration file 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mango-deployment
  labels:
    app: mango
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mango
  template:
    metadata:
      labels:
        app: mango
    spec:
      containers:
      - name: mangodb
        image: mango:5.0
        ports:
        - containerPort: 27017

```

The kind of this file is a deployment file.The are labels which acts as a tag for the pods.Used if you want to do operations on 
multiple pods at once.The template data is the configuration of the actual pod for example the image and the ports that you 
want to be opened etc .....So it has its own metadata and spec section.The configuration
for the deployment might be how many pods you want to be up.Labels can be the same while the name cant.We can label all the pod 
that were deployed using the same deployment file.The selector ,matchLabels is what kubernetes use to determine which pods belongs
to which deployment file.What is listed under matchlabels , should be the label of all pods of the deployment file.

If you want a replica of multiple databases you should use startfulset instead of deployments. 

We also want to use the secret file we created before to store the username and password for our database.
```
 env:
 - name:MANGO_INITDB_ROOT_USERNAME
   valueFrom:
    secretKeyRef:
      name:mango-secret
      key:mango-user 
 env:
 - name:MANGO_INITDB_ROOT_PASSWORD
   valueFrom:
    secretKeyRef:
      name:mango-secret
      key:mango-password
```
We set the name for the enviormental variable.Then we can just reference the value we set for password and username in the secrets
file by using the name of the file "mango-secret" and key for either the user or password.


## Create Service for Database
We will now attach a service to our mango database
```
apiVersion: v1
kind: Service
metadata:
  name: mango-service
spec:
  selector:
    app: mango 
  ports:
    - protocol: TCP
      port: 80
      targetPort: 27017
```
The selector is what the service will map to and the target port should always be linked to the container port ,which is 27017 that we set
before.The port is what the client will be connecting to. 

## Create a Deployment File for the Web-App

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  labels:
    app: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: "web image"
        ports:
        - containerPort: 1337

```
The image should be the image for your web-app.Then we need to configure the web app to be able to access the database 
```
env:
  - name:USER_NAME
    valueFrom:
      secretKeyRef:
        name:mango-secret
        key:mango-user
   - name:USER_NAME
    valueFrom:
      secretKeyRef:
        name:mango-secret
        key:mango-password
   - name:DB_URL
    valueFrom:
      configMapKeyRef:
        name: mango-config
        key: mango-url 
        
```
Add these enviormenetal variables to the web-app so it can connect to the database.It uses the secrets from the file we set before , as well
as the configmap we set before.


## Create a Service File for the Web App 
```
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type : NodePort
  selector:
    app: web 
  ports:
    - protocol: TCP
      port: 1337
      target port : 1337
      NodePort:30010
```
This service will now map to the web app.There is no configuration that is hard coded in this cluster.This makes it easy for us to 
make changes in the future.The "type:nodePort" are ports that will be opened on the nodes that external clients can connect to.They just
need to enter the node ip and port. The node port must be in the range of 30000-32767.

## Deploy the Pods
We just need to use the command 
```
kubectl get "file name"
```
for each yaml file we created.The configmap and secrets pods needs to be created first because other pods reference these two pods.


## Kubernetes command lines
Simple commands you can run to interact with the kubernetes cluster your just made
```
kubectl get all
```
Gives you everything that is running right now.

```
kubectl get "resources"
```
You can put things such as configmap ,secrets .pods over resources.
```
kubectl describe "resources"
```
Gives you detailed information of the resource.






