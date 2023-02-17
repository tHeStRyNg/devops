# Documentation for Task1

So to Dockerize a Python backend and React frontend, consider creating a Dockerfile for each component and a docker-compose file to orchestrate the containers.

BreakDown Structure of the project:

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

To achieve the above we started by creating the Dockerfile for the Python backend with the following logic and mapping the missing requirements:

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
```
psutil==5.9.4
Flask==2.2.2
Flask-Cors==3.0.10
jsonify==0.5
gunicorn==20.1.0
meinheld==1.0.2
```