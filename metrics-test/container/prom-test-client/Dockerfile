FROM python:3-alpine

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

COPY requirements.txt /usr/src/app
RUN pip install --no-cache-dir -r requirements.txt

COPY test-client.py /usr/src/app

EXPOSE 9877

ENTRYPOINT [ "python3", "-u", "test-client.py" ]
