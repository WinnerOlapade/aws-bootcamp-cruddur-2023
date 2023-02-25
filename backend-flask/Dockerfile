FROM python:3.10-slim-buster

# make a new cfolder inside the container
WORKDIR /backend-flask

# copy requirements.txt from folder to the folder inside the container
COPY requirements.txt requirements.txt

# run the command (python libraries used for the app)
RUN pip3 install -r requirements.txt

# copy all from present directory outside continer to the present directory in the container
COPY . .

# set environment variables inside container
ENV FLASK_ENV=development

# 
EXPOSE ${PORT}

# "python3 -m flask run --host=0.0.0.0 --port=4567" run this cmd in run_cmd.sh
CMD [ "./run_cmd.sh"]