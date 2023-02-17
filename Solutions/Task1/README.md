# Documentation for Task1

## Requirements (based on https://github.com/tHeStRyNg/devops/tree/master/DevOps-Assignment)
## Task 1 - Dockerize the Application

The first task is to dockerise this application - as part of this task you will have to get the application to work with Docker and Docker Compose. 
You are expected to get this app to work with UWSGI or Gunicorn and serve the react frontend through Nginx. 

The React container should also perform `npm build` every time it is built.

Hint/Optional - Create 3 separate containers. 1 for the backend, 2nd for the proxy and 3rd for the react frontend.

It is expected that you create another small document/walkthrough or readme which helps us understand your thinking process behind all of the decisions you made. 

The only strict requirement is that the application should spin up with `docker-compose up --build` command. 

You will be evaluated based on the
* best practices 
* ease of use
* quality of the documentation provided with the code


## Solution
So to Dockerize a Python backend and React frontend, consider creating a Dockerfile for each component and a docker-compose file to orchestrate the containers.

BreakDown Structure of the Backend, Frontend and NGinx containers:

```
├── Task1
│   ├── backend
│   │   ├── app
│   │   │   ├── app.py
│   │   │   └── requirements.txt
│   │   └── Dockerfile
│   ├── docker-compose.yml
│   ├── frontend
│   │   ├── Dockerfile
│   │   └── sys-stats
│   │       ├── npm-debug.log
│   │       ├── package.json
│   │       ├── public
│   │       │   ├── favicon.ico
│   │       │   ├── index.html
│   │       │   ├── logo192.png
│   │       │   ├── logo512.png
│   │       │   ├── manifest.json
│   │       │   └── robots.txt
│   │       ├── README.md
│   │       ├── src
│   │       │   ├── App.css
│   │       │   ├── App.js
│   │       │   ├── App.test.js
│   │       │   ├── index.css
│   │       │   ├── index.js
│   │       │   ├── logo.svg
│   │       │   ├── reportWebVitals.js
│   │       │   └── setupTests.js
│   │       └── yarn.lock
│   ├── nginx
│   │   └── default.conf
│   └── README.md
```

##  Backend Container
To achieve the above we started by creating the Dockerfile for the Python backend with the following logic and mapping the missing requirements adding gunicorn running on port 5000 as requested.
This uses psutils mainly to gather system stats (CPU and RAM) in a very fast way.

```
# Dockerfile
FROM tiangolo/meinheld-gunicorn:python3.8
WORKDIR /app
COPY app .
RUN pip install --no-cache-dir -r requirements.txt
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```
and the requirements dependencies:

```
# Python PIP requirements:
psutil==5.9.4
Flask==2.2.2
Flask-Cors==3.0.10
jsonify==0.5
gunicorn==20.1.0
meinheld==1.0.2
```

##  Frontend Container
The Frontend Container is based on NPM so we used alpine node 14:18-1 lts and just set it to npm start on port 3000 as follows below.
Important to denote that this service provides the "UI" or the "look and Feel" to display the stats being gathered from the backend node on port 5000.

```
FROM node:14.18.1-alpine
WORKDIR /app
COPY sys-stats .
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

##  NGinx Container

The NGinx container is the one that will be serving requests from port 80 proxying reverse to port 3000 of the frontend.
With this scope in mind i've just created a nginx defaults config file and setup the proxy reverse config to server from the frontend or node server and being created using docker compose on image nginx:latest as follows.


```
server {
  listen 80;
  server_name localhost;

  location / {
    proxy_pass http://frontend:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}

```

As you can see we are serving on port 80 of nginx service the content of Node frontend from 3000 with this one liner:
```
   proxy_pass http://frontend:3000;

```

# Docker Compose Container Orchestration

To tie all these togheter we used, as requested, docker-compose for the orchestration of all 3 container created above (backend, backend, nginx).
Also we have created a dependency on nginx to start dependant on the frontend which its suposed to serve.

```
version: '3'

services:
  backend:
    build: ./backend
    ports:
      - "5000:5000"
  frontend:
    build: ./frontend
    command: npm start
    ports:
      - "3000:3000"
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - frontend
```

## Conclusion
We've successfully dockerized this application and below you can find the build start and execution results

### 1 - Build
![build](https://user-images.githubusercontent.com/118682909/219665465-92bd2234-d5c9-41c3-a71b-69a76f0ccb96.png)

### 2 - Start
![start](https://user-images.githubusercontent.com/118682909/219665493-e305ad19-1ad8-4fd8-9c72-ec662b741cf2.png)

### 3 - Run
![execution](https://user-images.githubusercontent.com/118682909/219665523-6862ca0c-e64c-419b-90fb-7ad80da84f1d.png)

## How to Deploy
this was done with a Ubuntu 18 LTS and Docker-ce with Compose and git
- Install docker ``` apt install docker-ce ```
- Install docker-compose ``` curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose $$ chmod +x /usr/local/bin/docker-compose ```
- Make sure firewalls ports are open inbound for 80, 3000 and 5000 so pay attention if you need to rectify firewall rules ```ufw allow 30, 3000, 5000```
- Git pull this repo ``` git pull https://github.com/tHeStRyNg/devops.git ```
- Go to devops/Solutions/Task1 folder and exec the following ```docker-compose up --build ```

This will spin the 3 containers up then open your browser http://<REMOTE/LOCAL IP> and you will be able to see the react logo as mentioned above on Conclusion step 3.
