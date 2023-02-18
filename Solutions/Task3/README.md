## Requirements Definition for Task 3
#### Task 3 - Get it to work with Kubernetes (minikube)

- Next step is completely separate from step 2. Go back to the application you built in Stage 1 and get it to work with Kubernetes.
- Separate out the two containers into separate pods, communicate between the two containers, add a load balancer (or equivalent), expose the final App over port 80 to the final user (and any other tertiary tasks you might need to do)
- Add all the deployments, services and volume (if any) yaml files in the repo.
- The only hard-requirement is to get the app to work with `minikube`


## Pre-Requirements
### Environment Setup
 So we setup a Cloud Instance with the following specs:
- OS - Ubuntu 18.04 LTS x64 (Long Term Support)
-- 12 vCPUs, 24576.00 MB RAM, 500 GB NVMe Storage
- Docker
- Kubernetes - minikube

### Architecture Logic Behind the below code

The architecture consists of three Kubernetes deployments, each running as a separate pod.
- Python Backend Deployment: This deployment is responsible for running the Python backend code as a pod. It exposes its functionality over port 8000.
- React Frontend Deployment: This deployment is responsible for running the React frontend code as a pod. It exposes its functionality over port 3000.
- Nginx Deployment: This deployment is responsible for running the Nginx web server as a pod. It is configured to load balance traffic between the backend and frontend pods. It exposes its functionality over port 80.

In addition to the three deployments, there are three Kubernetes services that enable communication between the pods.
- Backend Service: This service allows the frontend pod to communicate with the backend pod over port 8000.
- Frontend Service: This service allows the Nginx pod to communicate with the frontend pod over port 3000.
- Nginx Service: This service is responsible for exposing the Nginx pod over port 80 to the outside world.
Finally, there is a Kubernetes ConfigMap that defines the Nginx configuration for serving the frontend and proxying requests to the backend. The ConfigMap is mounted as a volume in the Nginx pod and used to configure the Nginx web server.

Note:
The Docker and Kubernetes Registry configuration which holds the containers/pods images used in this example is not part of this exercise.

### Kubernetes Minikube and Kubectl Setup

So we install Kubernetes minikube as follows below. 
Note:
Important to denote that these settings are for dev environment only 
and to achieve this exercise in a faster timely manner as for best practices minikube shoudnt be run as root user.

#### Bash Script to automate the instalation
```
#!/bin/bash
# Description : Automation for setting up Kubernetes minikube
# Author: Nuno Miguel Tavares - stryng@gmail.com

apt-get update -y
apt-get upgrade -y
apt-get install curl -y
apt-get install apt-transport-https virtualbox virtualbox-ext-pack -y
wget https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
cp minikube-linux-amd64 /usr/local/bin/minikube
chmod 755 /usr/local/bin/minikube
minikube version
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
kubectl version -o json
minikube start --driver=docker --force
eval $(minikube docker-env)
minikube start --vm-driver="virtualbox" --insecure-registry="$REG_IP":80
```

## Initial Code Creation

To deploy the Python backend and React frontend as separate pods on Kubernetes, and expose them over port 80 using a loadbalancer, we will use the following Kubernetes configuration:

### Backend Deployment Config

First, we create a file backend-deployment.yaml with the following content:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    app: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: app
        image: <backend-image>
        command: ["gunicorn", "app:app", "--bind", "0.0.0.0:8000", "--workers", "4", "--worker-class", "gevent", "--log-level", "info"]
        ports:
        - containerPort: 8000

```

We will replace <backend-image> with the name of our backend Docker image from Task1.

### Frontend Deployment Config

First, we create a file frontend-deployment.yaml with the following content:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: app
        image: <frontend-image>
        command: ["npm", "start"]
        env:
        - name: REACT_APP_API_URL
          value: "http://backend:8000"
        ports:
        - containerPort: 3000

```

Like Before, we will replace <frontend-image> with the name of our previously created frontend Docker image from Task1.
This configuration defines a deployment for the frontend, with two replicas, and a container running npm to serve the React application on port 3000.

### LoadBalancer Deployment Config

Next, we create a file nginx-deployment.yaml with the following content:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: nginx-config
        configMap:
          name: nginx-config

```
This configuration defines a deployment for Nginx, with one replica, and a container running the Nginx image from Docker Hub, serving traffic on port 80.

### NGINX Service Configuration

Next, we create a file nginx-service.yaml with the following content:

```
apiVersion: v1
kind: Service
metadata:
  name: backend
  labels:
    app: backend
spec:
  selector:
    app: backend
  ports:
  - name: http
    port: 8000
    targetPort: 8000

```

This configuration defines a service for the backend deployment, allowing it to be accessed by other pods in the cluster.

### Frontend Service Configuration
Next, we create a file named frontend-service.yaml with the following content:

```
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  selector:
    app: frontend
  ports:
  - name: http
    port: 3000
    targetPort: 3000
```
This configuration defines a service for the frontend deployment, allowing it to be accessed by other pods in the cluster.


### NGINX Configuration
Next, we create a file named nginx-config.yaml with the following content:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  default.conf: |
    server {
        listen       80;
        server_name  localhost;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
            try_files $uri /index.html;
        }

        location /api/ {
            proxy_pass         http://backend:8000/;
            proxy_redirect     off;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
        }
    }
```
This configuration defines a ConfigMap for Nginx, with a custom default.conf file defining the Nginx configuration for the frontend and the backend.

### Kubernetes minikube cmds that might be useful
Before applying the above configuration make sure minikube is ready to create loads and all is operational

```
kubectl version -o json
kubectl config view
kubectl cluster-info
kubectl get nodes
kubectl get pod
kubectl get svc
kubectl get deployments
minikube ip
```

### Apply all the above Configurations
Finally, we apply all the configurations using the following commands:

```
kubectl apply -f backend-deployment.yaml
kubectl apply -f backend-service.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
kubectl apply -f nginx-config.yaml
```
This will deploy the backend, frontend, and Nginx pods, as well as the corresponding services and the Nginx ConfigMap. 
The frontend and backend pods will communicate through the backend service, and the Nginx pods will load balance traffic between the frontend and backend pods over port 80. 
The final app will be exposed over port 80 to the end user.

### Summary - Conclusion

We check existing nodes running and if minikube was accessible as follows:

```
~# kubectl get nodes
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   20h   v1.26.1

~# kubectl get pod
No resources found in default namespace.

~:/opt/git/devops/Solutions/Task3# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6s

~:/opt/git/devops/Solutions/Task3# kubectl get deploy
No resources found in default namespace.
```

And then we apply the configuration we did as follows.

```
$:/opt/git/devops/Solutions/Task3# kubectl apply -f backend-service.yaml
service/backend created

$:/opt/git/devops/Solutions/Task3# kubectl apply -f frontend-deployment.yaml
deployment.apps/frontend created

$:/opt/git/devops/Solutions/Task3# kubectl apply -f frontend-service.yaml
service/frontend created

$:/opt/git/devops/Solutions/Task3# kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx created

$:/opt/git/devops/Solutions/Task3# kubectl apply -f nginx-service.yaml
service/nginx created

$:/opt/git/devops/Solutions/Task3# kubectl apply -f nginx-config.yaml
configmap/nginx-config created

$:/opt/git/devops/Solutions/Task3# kubectl get svc
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
backend      ClusterIP      10.106.31.59     <none>        8000/TCP       7s
frontend     ClusterIP      10.99.51.119     <none>        3000/TCP       6s
kubernetes   ClusterIP      10.96.0.1        <none>        443/TCP        91s
nginx        LoadBalancer   10.104.111.180   <pending>     80:30723/TCP   6s

$:/opt/git/devops/Solutions/Task3# kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
backend    1/2     2            1           14s
frontend   1/2     2            1           14s
nginx      1/1     1            1           13s

$:/opt/git/devops/Solutions/Task3# kubectl get pod
NAME                        READY   STATUS           RESTARTS   AGE
backend-58495b5747-pktkr    1/1     Running             0       19s
backend-58495b5747-sm2cx    1/1     Running             0       19s
frontend-7db7b9b495-g5b98   1/1     Running             0       19s
frontend-7db7b9b495-mmsrq   1/1     Running             0       19s
nginx-54d8fc87b9-bssnz      1/1     Running             0       18s

```
We could expose the nginx service port using NodePort as follows but instead well "hack it"... so this is just info content.

```
kubectl expose deployment nginx --type=NodePort
```

So for this VSC "hack" we will check the port 80 with a reverse port forward using Terminal Ports on VSC as follows below.
PS --> we could also use curl to test the port (we'll skip).
 
![vcs](https://user-images.githubusercontent.com/118682909/219866362-c6ebaa27-dfdf-46d8-8536-df0da48b6bc8.PNG)
 
 Your default defined browser will open pn http://localhost:80 and the following is displayed.

![execution](https://user-images.githubusercontent.com/118682909/219866401-bd1529eb-69eb-4668-ace2-4adadf7c8e6a.png)

### Conclusion
Done application running with 2 backend replicas, 2 frontend replicas and 1 loadbalancer on Kubernetes Minikube.
