# FlaskApp

## Requirements

- Docker
- Minikube (or a cloud hosted cluster)
- Kubectl

## Notes

A flask api app that connects to a SQL server all installed, after building the image, in one command. 

This is mostly to be done locally and using minikube but you can apply helm onto any cluster provided the image is uploaded in a location where it can be pulled. For this to work, ensure you have modified the `flaskapp-deployment` file:

```image: flask-app
   imagePullPolicy: Never
```

So that it can pull the image.


## Building The Image

If using minikube, this may be needed so that minikube can locate the image locally.

`eval $(minikube docker-env)`

To build the image, simply go to the `app` directory and type in

```docker build -t flask-app ./app```

## Apply Via Helm

Install everything else via helm! Helm will install

- SQL Deployment
- PersistentVolume
- Secrets
- FlaskApp Deployment

Simply run

```helm install flask-app flaskapp/```

## Start

### Configure SQL Databse and Users Table

If starting from scratch, we would need to setup SQL with the proper database and users table. Access SQL by running this command:


```kubectl run -it --rm --image=mysql --restart=Never mysql-client -- mysql --host mysql --password=helloworld```

Afterwards run the SQL command

`CREATE DATABASE flaskapi;
USE flaskapi;
CREATE TABLE users(user_id INT PRIMARY KEY AUTO_INCREMENT, user_name VARCHAR(255), user_email VARCHAR(255), user_password VARCHAR(255));`

### Minikube Service

To get the service started, type in

```minikube service flaskapp-service```

And this should show up

```

|-----------|------------------|-------------|---------------------------|
| NAMESPACE |       NAME       | TARGET PORT |            URL            |
|-----------|------------------|-------------|---------------------------|
| default   | flaskapp-service |        5000 | http://192.168.49.2:31184 |
|-----------|------------------|-------------|---------------------------|
```

> If there are issues with connections, this may be required as a minikube command `minikube start --cni=flannel`


### For Use

Take note of the `URL`. Make sure you have the right one from when starting up minikube service (or whichever cluster you are using)

#### Create User

Command
```curl -H "Content-Type: application/json" -d '{"name": "random", "email": "r@r.com", "pwd": "random"}' http://192.168.49.2:31184/create```

Output
```"User created successfully!"```


#### Get Users

Command
```curl http://192.168.49.2:31184/users```

Output
```[[1,"random","r@r.com","random"]]```

### Get Specific User

Command
```curl http://192.168.49.2:31184/user/1```

Output
```[1,"random","r@r.com","random"]```

### Update User

Command
```curl -H "Content-Type: application/json" -d '{"name": "no_longer_random", "email": "r@r.com", "pwd": "random", "user_id":"1"}' http://192.168.49.2:31184/update```


Output
```[1,"no_longer_random","r@r.com","random"]```

#### Delete Users
```curl http://192.168.49.2:31184/delete/1```

Output
```"User deleted successfully!"```



## Deployments and Other Notes

The best way for continuous delivery in the case of using helm charts is via ArgoCD. We can leverage the usage of Github Actions to build the image and deploy it to dockerhub where the updated version of the image will remain. Everytime a push is made, Github Actions will build and upload a new image. We then move all the image tag values from the deployment files onto the `values.yaml` file.

```image: "{{ .Values.image.tag }}"```

Once that is done, whenever we do our build, we can also update the value in the `values.yaml` file and configure ArgoCD to immediately deploy the new image to the cluster. This is better if we have kubernetes hosted on something like AWS EKS as well. This way we can leverage the use of ArgoCD to deploy to multiple environments/clusters.

If we are to use AWS EKS or to host kubernetes in the cloud, we should consider the usage of Terraform to build all infrastructure to follow the IaC methodology.

In short, use Terraform to build the infrastructure to host our kubernetes cluster in AWS EKS. Have Github Actions as our CI tool in building and uploading the images and have ArgoCD perform the deployments automatically.
