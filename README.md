# Kubernetes Web Deployment
In this guide i will go over how to use Kubernetes to deploy a web app with a mangodb database attached.I will use Minikube to set 
up a local cluster.I will create a config , service , and deployment file using yaml.I used Kubernetes documentation for this project as well
as this youtube video https://www.youtube.com/watch?v=s_o8dwzRlu4&t=1138s.

## Installation 
Minikube can run either on a virtual machine or as a docker container.I will be using windows to install Minikube.First you would have to 
download Docker Desktop for windows.Then you need to download Minikube either by downloading the exe file or Powershell.Then run the command
minikube start. Minikube CLI is for starting and deleteing the cluster 


## Create a config map
I qill create a configMap to map the web app to the database
```
apiVersion: v1
kind: ConfigMap
metadata:
  name : mango-config-map
data:
  mongo-url : mango-service 
```
The name of this is called mango-config-map .The data attribute maps to the mango database that we will create later.We will attach a service
to the mangodb database.

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
    app: nginx
spec:
  replicas: 3
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


## Create Service 
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
        - containerPort: 27017

```




