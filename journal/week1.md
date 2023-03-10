# Week 1 — App Containerization

## Containerize Backend

### Add Dockerfile to backend folder
I created a dockerfile in the [backend-flask](backend-flask/Dockerfile) folder which contains:

```dockerfile
FROM python:3.10-slim-buster
WORKDIR /backend-flask
COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txtdirectory in the container
COPY . .
ENV FLASK_ENV=development
EXPOSE ${PORT}
CMD ./run_cmd.sh
```

I added "run_cmd" script which contains the command which Dockerfile CMD will execute (this becomes useful when there are several lines of command to run).
Content of [run_cmd.sh](backend-flask/run_cmd.sh):

```sh
#!/bin/bash
python3 -m flask run --host=0.0.0.0 --port=4567
```

### Build Backend Container
I built the backend container image (with container name "backend-flask") with the command:

```sh
docker build -t  backend-flask ./backend-flask
```
### Run Backend Container
I then run the container (using the container image built) and passing in the environmental variables (Backend_url and Frontend_url) in the same command which also maps port 4567 to 4567:

```sh
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
```


## Containerize Frontend

### Run NPM install
I executed the ` npm i ` command while in the frontend-react folder before building the container.

```sh
cd frontend-react-js
npm i
```

### Create Dockerfile to Frontend folder
I created a Dockerfile in [frontend-react-js](frontend-react-js/Dockerfile) folder with content:

```dockerfile
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

```sh
docker build -t frontend-react-js ./frontend-react-js
```

### Run Frontend Container
I run the frontend container buit with the command:

```sh
docker run -p 3000:3000 -d frontend-react-js
```



## Multiple Containers

### Create docker-compose file
I created a [docker-compose.yml](docker-compose.yml) file at the root directory of my repository (to connect both the frontend and the backend of the app together) with the content:

```yml
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

networks: 
  internal-network:
    driver: bridge
    name: cruddur   
```

### Run Docker-compose
I run the docker-compose.yml with the command:

```
docker-compose up
```

### Note
each time i run a container I unlock the ports mapped to the container and test by clicking on the links provided in the gitpod -> ports section of the terminal to check if properly configured.

** Insert Picture here **



## Add Notifications to Backend (Flask)

### Add notifications to Openapi file
I edited [openapi-3.0.yml](backend-flask/openapi-3.0.yml) present in the backend-flask folder of my repository and added block of code for notifications section of the backend.
 
 ```yml
   /api/activities/notifications:
    get:
      description: 'Return a feed of activities for all those I follow'
      tags:
        - activities
      parameters: []
      responses:
        '200':
          description: 'Returns an array of activities'
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Activity'
 ```
 
### Added Notifications to app.py
I added notifications to the [app.py](backend-flask/app.py) also present in the backend-flask folder. 
Added the following code to the corresponding block:
 
```python
 from services.notifications_activities import *
```
 
```python
 @app.route("/api/activities/notifications", methods=['GET'])
def data_notifications():
  data = NotificationsActivities.run()
  return data, 200
```

### Added notifications_activities to Backend services
I created a new file [notifications_activities.py](backend-flask/services/notifications_activities.py) and added the code:

```python
from datetime import datetime, timedelta, timezone
class NotificationsActivities:
  def run():
    now = datetime.now(timezone.utc).astimezone()
    results = [{
      'uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
      'handle':  'Winner Olapade',
      'message': 'Tis is going great! ',
      'created_at': (now - timedelta(days=2)).isoformat(),
      'expires_at': (now + timedelta(days=5)).isoformat(),
      'likes_count': 5,
      'replies_count': 1,
      'reposts_count': 0,
      'replies': [{
        'uuid': '26e12864-1c26-5c3a-9658-97a10f8fea67',
        'reply_to_activity_uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
        'handle':  'Worf',
        'message': 'This post has no honor!',
        'likes_count': 0,
        'replies_count': 0,
        'reposts_count': 0,
        'created_at': (now - timedelta(days=2)).isoformat()
      }],
    },
    ]
    return results
```



## Add notifications to Frontend (React)

### Add notifications to app.js
I added the following code to the correcsponding blocks of the [app.js](frontend-react-js/src/App.js) file present in the frontend-react-js folder in my repository:

```js
import NotificationsFeedPage from './pages/NotificationsFeedPage';
```

```js
{
    path: "/notifications",
    element: <NotificationsFeedPage />
  },
```

### Add notification pages
I created 2 files (NotificationsFeedPage.js, NotificationsFeedPage.css) in  "pages" folder of "src" folder of the frontend-react-js folder (frontend-react-js -> src -> pages).

I added the following code to "[NotificationsFeedPage.js](frontend-react-js/src/pages/NotificationsFeedPage.js)":

```js
import './NotificationsFeedPage.css';
import React from "react";

import DesktopNavigation  from '../components/DesktopNavigation';
import DesktopSidebar     from '../components/DesktopSidebar';
import ActivityFeed from '../components/ActivityFeed';
import ActivityForm from '../components/ActivityForm';
import ReplyForm from '../components/ReplyForm';

// [TODO] Authenication
import Cookies from 'js-cookie'

export default function NotificationsFeedPage() {
  const [activities, setActivities] = React.useState([]);
  const [popped, setPopped] = React.useState(false);
  const [poppedReply, setPoppedReply] = React.useState(false);
  const [replyActivity, setReplyActivity] = React.useState({});
  const [user, setUser] = React.useState(null);
  const dataFetchedRef = React.useRef(false);

  const loadData = async () => {
    try {
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/activities/notifications`
      const res = await fetch(backend_url, {
        method: "GET"
      });
      let resJson = await res.json();
      if (res.status === 200) {
        setActivities(resJson)
      } else {
        console.log(res)
      }
    } catch (err) {
      console.log(err);
    }
  };

  const checkAuth = async () => {
    console.log('checkAuth')
    // [TODO] Authenication
    if (Cookies.get('user.logged_in')) {
      setUser({
        display_name: Cookies.get('user.name'),
        handle: Cookies.get('user.username')
      })
    }
  };

  React.useEffect(()=>{
    //prevents double call
    if (dataFetchedRef.current) return;
    dataFetchedRef.current = true;

    loadData();
    checkAuth();
  }, [])

  return (
    <article>
      <DesktopNavigation user={user} active={'notifications'} setPopped={setPopped} />
      <div className='content'>
        <ActivityForm  
          popped={popped}
          setPopped={setPopped} 
          setActivities={setActivities} 
        />
        <ReplyForm 
          activity={replyActivity} 
          popped={poppedReply} 
          setPopped={setPoppedReply} 
          setActivities={setActivities} 
          activities={activities} 
        />
        <ActivityFeed 
          title="notifications" 
          setReplyActivity={setReplyActivity} 
          setPopped={setPoppedReply} 
          activities={activities} 
        />
      </div>
      <DesktopSidebar user={user} />
    </article>
  );
}
```

I added the following code to "[NotificationsFeedPage.css](frontend-react-js/src/pages/NotificationsFeedPage.css)":

```css
article {
    display: flex;
    flex-direction: row;
    justify-content: center;
  }
```



## Add Postgres and DynamoDB Local

### Adding Postgres to docker-compose
I added postgres database to the docker-compose file in the root folder of my repository with the code:

```yml
services:
  db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
volumes:
  db:
    driver: local
```

I also added Postgres client into my Gitpod by adding the following to the gitpod.yml file also present in root directory:

```yml
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```

### Adding DynamoDB Local to docker-compose
I added DynamoDB local to the existing docker-compose.yml file:

```yml
services:
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
```

### DynamoDB Local tryout
I also tried to run the DynamoDB Local using the following steps from https://github.com/100DaysOfCloud/challenge-dynamodb-local



## Homework Challenge

### Run the dockerfile CMD as an external script
- I added an external script [run_cmd.sh](backend-flask/run_cmd.sh) with the command:

```sh
#!/bin/bash
python3 -m flask run --host=0.0.0.0 --port=4567
```

- I added the path of the script as the Dockerfile CMD
 
```dockerfile
CMD [ "./run_cmd.sh"]
```

### Push and tag a image to DockerHub
- I already have a dockerHub account so I logged into Gitpod cli using:

  ```
  docker login
  ```
  
- I created aand tagged a container image using the backend-flask dockerfile:

  ```
  docker tag backend-flask onyankupon/aws-cruddur-bootcamp:1.0
  ```
  
- I pushed the taggged container to DockerHub using the command:
  
  ```
  docker push onyankupon/aws-cruddur-bootcamp:1.0
  ```
  
To pull image from repository try:

```
docker pull onyankupon/aws-cruddur-bootcamp:1.0
```
 

###  Install Docker on your localmachine
I already have docker installed on my Local machine

### Launch an EC2 instance that has Docker installed & pull a container
- I launched an EC2 instance, connected to the EC2 instance, and Installed Docker using the [Official Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-container-image.html)

- then pulled the container image pushed to my Docker repository using the command:

  ```
  docker pull onyankupon/aws-cruddur-bootcamp:1.0
```

