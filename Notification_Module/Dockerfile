FROM python:3-slim

WORKDIR /app

COPY requirements.txt ./

RUN python -m pip install --no-cache-dir -r requirements.txt

COPY ./amqp_setup.py ./notifications.py ./

EXPOSE 4999

CMD [ "python", "notifications.py" ]