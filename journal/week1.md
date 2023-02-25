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
CMD [ "./run_cmd.sh"]
```

I added "run_cmd" script which contains the command which Dockerfile CMD will execute (this becomes useful when there are several lines of command to run).
Content of [run_cmd.sh](backend-flask/run_cmd.sh):

```sh
#!/bin/bash
python3 -m flask run --host=0.0.0.0 --port=4567
```

### Build Backend Container
I built the backend container image (with container name "backend-flask") with the command:

```
docker build -t  backend-flask ./backend-flask
```
### Run Backend Container
I then run the container (using the container image built) and passing in the environmental variables (Backend_url and Frontend_url) in the same command which also maps port 4567 to 4567:

```
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
```


## Containerize Frontend

### Run NPM install
I executed the ` npm i ` command while in the frontend-react folder before building the container.

```
cd frontend-react-js
npm i
```

### Create Dockerfile to Frontend folder
I created a Dockerfile in [frontend-react-js](frontend-react-js/Dockerfile) folder with content:

```
FROM node:16.18
ENV PORT=3000
COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```

### Build Frontend Container
I built the frontend container (with container name "frontend-react-js") using the command:

```
docker build -t frontend-react-js ./frontend-react-js
```

### Run Frontend Container
I run the frontend container buit with the command:

```
docker run -p 3000:3000 -d frontend-react-js
```
