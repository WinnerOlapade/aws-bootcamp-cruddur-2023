# Week 1 — App Containerization

## Containerize Backend

### Add Dockerfile to backend folder
I created a dockerfile in the [backend-flask](backend-flask/Dockerfile) folder which contains:

```
FROM python:3.10-slim-buster
WORKDIR /backend-flask
COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txtdirectory in the container
COPY . .
ENV FLASK_ENV=development
EXPOSE ${PORT}

# "python3 -m flask run --host=0.0.0.0 --port=4567" run this cmd in run_cmd.sh
CMD [ "./run_cmd.sh"]
```

I added run_cmd script which contains command which Dockerfile CMD will execute.
Content of [run_cmd.sh](backend-flask/run_cmd.sh):

```sh
#!/bin/bash
python3 -m flask run --host=0.0.0.0 --port=4567
```

