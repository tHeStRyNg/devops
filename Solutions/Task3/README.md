# Documentation for Task 3
## Task 3 - Get it to work with Kubernetes (minikube)

- Next step is completely separate from step 2. Go back to the application you built in Stage 1 and get it to work with Kubernetes.
- Separate out the two containers into separate pods, communicate between the two containers, add a load balancer (or equivalent), expose the final App over port 80 to the final user (and any other tertiary tasks you might need to do)
- Add all the deployments, services and volume (if any) yaml files in the repo.
- The only hard-requirement is to get the app to work with `minikube`

### Environment Setup
 So we setup a Cloud Instance with the following specs:
- OS - Ubuntu 18.04 LTS x64 (Long Term Support)
-- 12 vCPUs, 24576.00 MB RAM, 500 GB NVMe Storage
- Docker
- Kubernetes - minikube

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
minikube start --force
```

###