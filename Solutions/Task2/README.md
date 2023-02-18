## Requirements Definition for Task 2
### Task 2 - Deploy on Cloud

### Deployment of Task 1 Containers in AWS Kubernetes or EKS

We decided that we will deploy the Python backend and React frontend on AWS EKS.
To deploy the docker-compose file we created on task 1 on AWS EKS, we would need to do the following steps:

1 - Create an EKS cluster using the AWS Management Console or the AWS CLI.

2 - Install and configure the AWS CLI on our local machine.

3 - Create an Amazon ECR repository and push the Docker images for the backend and frontend services to the repository.

4 - Create a Kubernetes deployment file, which describes how the Docker containers should be deployed in the EKS cluster. 
This is its deployment file:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: backend
          image: <your-ecr-repo>/backend:latest
          ports:
            - containerPort: 5000
        - name: frontend
          image: <your-ecr-repo>/frontend:latest
          command: ["npm", "start"]
          ports:
            - containerPort: 3000
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-config
              mountPath: /etc/nginx/conf.d/default.conf
      volumes:
        - name: nginx-config
          configMap:
            name: nginx-config
      imagePullSecrets:
        - name: <your-ecr-registry-creds>
```

Note that you will need to replace ```<your-ecr-repo>``` with the name of your ECR repository and ```<your-ecr-registry-creds>``` with the name of your ECR registry credentials.

5 - Create a Kubernetes service file, which exposes the backend and frontend containers to the internet.
This is the service file

```
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
    - name: backend
      port: 5000
      targetPort: 5000
    - name: frontend
      port: 3000
      targetPort: 3000
    - name: nginx
      port: 80
      targetPort: 80
```

6 - Apply the deployment and service files to the EKS cluster using the ```kubectl apply``` command:

```
kubectl apply -f <path-to-deployment-file>
kubectl apply -f <path-to-service-file>
```

7 - Wait for the LoadBalancer service to be created. You can check the status of the service using the kubectl get service command:

```
kubectl get service my-app
```

8 - Access your application using the external IP address of the LoadBalancer service, which you can find using the ```kubectl get service``` command.

That's it! The application is now running on AWS EKS.
