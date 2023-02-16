FROM python:3.10.8-alpine
RUN apk update && apk add zlib-dev linux-headers py3-psutil bzip2-dev xz-dev cargo && apk add --no-cache build-base
WORKDIR /app
COPY app .
RUN pip install --upgrade pip && pip install -r requirements.txt
RUN python app.py
COPY . .
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]