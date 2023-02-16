FROM python:3.10.8-alpine
RUN apk update && apk add zlib-dev && apk add --no-cache build-base
RUN apk add bzip2-dev
RUN apk add xz-dev
RUN apk add cargo
RUN apk add rust
WORKDIR /usr/src/app
COPY app.py requirements.txt .
RUN pip install -r requirements.txt
RUN python app.py
COPY . .
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]