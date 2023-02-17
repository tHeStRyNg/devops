FROM tiangolo/meinheld-gunicorn:python3.8
WORKDIR /app
COPY app .
RUN pip install --no-cache-dir -r requirements.txt
#CMD [ "python", "./app.py" ]
#RUN python app.py
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]