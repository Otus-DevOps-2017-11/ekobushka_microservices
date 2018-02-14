FROM python:alpine3.6

# RUN pip install flask pymongo

WORKDIR /app
# ADD . /app
COPY . /app

RUN pip install -r requirements.txt

ENV POST_DATABASE_HOST post_db
ENV POST_DATABASE posts

CMD ["python3", "post_app.py"]
