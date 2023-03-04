# Week 1 — App Containerization

## References

Good Article for Debugging Connection Refused
https://pythonspeed.com/articles/docker-connection-refused/


## VSCode Docker Extension

Docker for VSCode makes it easy to work with Docker

https://code.visualstudio.com/docs/containers/overview

> Gitpod is preinstalled with theis extension

## Containerize Backend
Prior to containerization I ran the application locally to confirm it was working 

### Run Python

```
pip install -r requirements.txt

```

```sh
cd backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
python3 -m flask run --host=0.0.0.0 --port=4567
cd ..
```

- I set the enviroment and did and imstalled requirements using pip
- I umlocked the port 4567 in vscode and open up the link on the browser 
- I initially got a 404 error however I got a json file after appending the url to `/api/activities/home`

![data](/assets/Capture_pip_requirement_.PNG)
![data](/assets/Capture_confirm_cruuder_1.PNG)

### Add Dockerfile

Create a file here: `backend-flask/Dockerfile`

```dockerfile
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```
![data](/assets/Capture_build_docker_images_2.PNG)

### Build Container

```sh
docker build -t  backend-flask ./backend-flask
```
![data](/assets/Capture_build_docker_images_2.PNG)

### Run Container

- Running the images with the enviroment not set threw an error  
- Set the enviroemnt variables 

![data](/assets/Capture_enviroment_variable_not_set_3.PNG)

![data](/assets/Capture_set_env_vars_from_command_line_4.PNG)

Run 
```sh
docker run --rm -p 4567:4567 -it backend-flask
FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
docker run --rm -p 4567:4567 -it  -e FRONTEND_URL -e BACKEND_URL backend-flask
unset FRONTEND_URL="*"
unset BACKEND_URL="*"
```

![data](/assets/Capture_docker_ps_docker_images_5.PNG)

Run in background
```sh
docker container run --rm -p 4567:4567 -d backend-flask
```

Return the container id into an Env Vat
```sh
CONTAINER_ID=$(docker run --rm -p 4567:4567 -d backend-flask)
```

> docker container run is idiomatic, docker run is legacy syntax but is commonly used.

### Get Container Images or Running Container Ids

```
docker ps
docker images
```
![data](/assets/Capture_7.PNG)

### Send Curl to Test Server

```sh
curl -X GET http://localhost:4567/api/activities/home -H "Accept: application/json" -H "Content-Type: application/json"
```
- This threw a cors error that we have to fix later like Andrew said. 
![data](/assets/Capture_cors_error_8.PNG)

### Check Container Logs

```sh
docker logs CONTAINER_ID -f
docker logs backend-flask -f
docker logs $CONTAINER_ID -f
```

###  Debugging  adjacent containers with other containers

```sh
docker run --rm -it curlimages/curl "-X GET http://localhost:4567/api/activities/home -H \"Accept: application/json\" -H \"Content-Type: application/json\""
```

busybosy is often used for debugging since it install a bunch of thing

```sh
docker run --rm -it busybosy
```

### Gain Access to a Container

```sh
docker exec CONTAINER_ID -it /bin/bash
```

> You can just right click a container and see logs in VSCode with Docker extension

### Delete an Image

```sh
docker image rm backend-flask --force
```

> docker rmi backend-flask is the legacy syntax, you might see this is old docker tutorials and articles.

> There are some cases where you need to use the --force

### Overriding Ports

```sh
FLASK_ENV=production PORT=8080 docker run -p 4567:4567 -it backend-flask
```

> Look at Dockerfile to see how ${PORT} is interpolated

## Containerize Frontend

## Run NPM Install

We have to run NPM Install before building the container since it needs to copy the contents of node_modules

```
cd frontend-react-js
npm i
```
![data](/assets/Capture_npm_install_9.PNG)

### Create Docker File

Create a file here: `frontend-react-js/Dockerfile`

```dockerfile
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```
### Build Container

```sh
docker build -t frontend-react-js ./frontend-react-js
```

### Run Container

```sh
docker run -p 3000:3000 -d frontend-react-js
```
![data](/assets/Capture_build_docker_images_frontend_10.PNG)
![data](/assets/Capture_ports_open_11.PNG)

## Multiple Containers

### Create a docker-compose file

Create `docker-compose.yml` at the root of your project.

```yaml
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

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```
![data](/assets/Capture_docker_compose_up_13.PNG)
![data](/assets/Capture_docker_compose_ps_14.PNG)
![data](/assets/Capture_docker_compose_ls_images_15.PNG)

- I ensured that the padlock on ports 3000 and 4567 are open in the ports session, then open the link with port 300 in the browser.
I encountered the CORS (Cross Origin Resource Sharing) error encountered during the livestream. I checked that the ports are correct in the docker-compose file, and ran the `docker compose -f "docker-compose.yml" up -d --build`.

![data](/assets/Capture_crudder_app_online_16.PNG)

### Created a notification feature for backend and frontend
### Added this in the openapi.yml file

      /api/activities/notifications:
    get:
      description: 'Return a feed of activity for all the people i follow'
      tags:
        - activities
        
      parameters: []
      responses:
        '200':
          description: Returns an array of activities
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Activity'

![openai](/assets/Capture_open_api_notifications_17.PNG)

### In the backend-flask/services directory created, I created a notifications_activities.py file and pasted the below:

    from datetime import datetime, timedelta, timezone
    class NotificationsActivities:
    def run():
        now = datetime.now(timezone.utc).astimezone()
        results = [{
        'uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
        'handle':  '3abuza',
        'message': 'I am a DevOps Engineer',
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

![notifications](/assets/Capture_notifications_py_18.PNG)

### In the frontend/src/pages directory, I added a new page NotificationsFeedPage.js file and pasted the following code

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
                title="Notifications" 
                setReplyActivity={setReplyActivity} 
                setPopped={setPoppedReply} 
                activities={activities} 
                />
            </div>
            <DesktopSidebar user={user} />
            </article>
        );
        }

![notification](/assets/Capture_notification_activities_js_19.PNG)

### in the frontend-react-js/src/App.js file, add the following:

    import NotificationsFeedPage from './pages/NotificationsFeedPage';

    {
       path: "/notifications",
       element: <NotificationsFeedPage />
    },

## Adding DynamoDB Local and Postgres

We are going to use Postgres and DynamoDB local in future labs
We can bring them in as containers and reference them externally

Lets integrate the following into our existing docker compose file:

### Postgres

```yaml
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

To install the postgres client into Gitpod

```sh
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```

### DynamoDB Local
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

Example of using DynamoDB local [https://github.com/100DaysOfCloud/challenge-dynamodb-local](https://github.com/100DaysOfCloud/challenge-dynamodb-local)

### Volumes
Directory volume mapping

    volumes: 
    - "./docker/dynamodb:/home/dynamodblocal/data"

### named volume mapping
    volumes: 
      - db:/var/lib/postgresql/data

    volumes:
      db:
        driver: local

### Challenge Dynamodb Local
Dynamodb Local emulates a Dynamodb database in your local envirmoment for rapid developement and table design interation

### Run Docker Local
    docker-compose up
> This is an alternative command to `docker compose -f "docker-compose.yml up -d --build`

![postgres_dynamodb](/assets/Capture_docker_compose_up_dynamodb_postgres_20.PNG)

### Create a table
    aws dynamodb create-table \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --table-class STANDARD
![postgres_dynamodb](/assets/Capture_dynamodb_create_table_22.PNG)
### List Tables
    aws dynamodb list-tables --endpoint-url http://localhost:8000

![postgres_dynamodb](/assets/Capture_dynamodb_list_table_23.PNG)

### Create an Item
    aws dynamodb put-item \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --item \
        '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}}' \
    --return-consumed-capacity TOTAL  
![postgres_dynamodb](/assets/Capture_dynamodb_put_item_24.PNG)
### Get Records

    aws dynamodb scan --table-name Music --query "Items" --endpoint-url http://localhost:8000
    
![postgres_dynamodb](/assets/Capture_dynamodb_get_item_25.PNG)

### Install postgresql
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev          

### Test Postgresql connection

![postgres_dynamodb](/assets/Capture_connect_to _postgres_29.PNG)
![postgres_dynamodb](/assets/Capture_list_databases_pgsql_30.PNG)
![postgres_dynamodb](/assets/Capture_connection_pgsql_client_32.PNG)

### Homework Challenges 

### Research best practices of Dockerfiles and attempt to implement it in your Dockerfile
### Best practices for writing Dockerfiles
Docker builds images automatically by reading the instructions from a Dockerfile.
A Dockerfile follows a specific format and set of instructions which you can find at [Dockerfile reference](https://docs.docker.com/engine/reference/builder/).
A Docker image consists of read-only layers each of which represents a Dockerfile instruction. The layers are stacked and each one is a delta of the changes from the previous layer as seen when building images for both backend and frontend app.

General guidelines and recommendations
1. Understand build context
  The docker build or docker buildx build commands build Docker images from a Dockerfile and a “context”.

  A build’s context is the set of files located at the PATH or URL specified as the positional argument to the build command:
    `docker build [OPTIONS] PATH` as used when building the images. docker build -t backend ./backend-flask/ 
   `./backend-flask` is he PATH specified as the positional argument to the build command.

2. Create ephemeral containers
  The image defined by your Dockerfile should generate containers that are as ephemeral as possible. Ephemeral means that the container can be stopped and destroyed, then rebuilt and replaced with an absolute minimum set up and configuration

3. Exclude with .dockerignore
   To exclude files not relevant to the build, without restructuring your source repository, use a .dockerignore file. This file supports exclusion patterns similar to .gitignore files.

4. Use multi-stage builds
   Multi-stage builds allow you to drastically reduce the size of your final image, without struggling to reduce the number of intermediate layers and files.

  Because an image is built during the final stage of the build process, you can minimize image layers by leveraging build cache.

  For example, if your build contains several layers and you want to ensure the build cache is reusable, you can order them from the less frequently changed to the more frequently changed. The following list is an example of the order of instructions:

  * Install tools you need to build your application

  * Install or update library dependencies

  * Generate your application

5. Don’t install unnecessary packages
   Avoid installing extra or unnecessary packages just because they might be nice to have. For example, you don’t need to include a text editor in a database image.

   When you avoid installing extra or unnecessary packages, you images will have reduced complexity, reduced dependencies, reduced file sizes, and reduced build times.

6. Pipe Dockerfile through stdin
   Docker has the ability to build images by piping a Dockerfile through stdin with a local or remote build context. Piping a Dockerfile through stdin can be useful to perform one-off builds without writing a Dockerfile to disk, or in situations where the Dockerfile is generated, and should not persist afterwards.

7. Build an image using a Dockerfile from stdin, without sending build context
    Use this syntax to build an image using a Dockerfile from stdin, without sending additional files as build context. The hyphen (-) takes the position of the PATH, and instructs Docker to read the build context, which only contains a Dockerfile, from stdin instead of a directory: `docker build [OPTIONS] -`
    The following example builds an image using a Dockerfile that is passed through stdin. No files are sent as build context to the daemon.


            docker build -t myimage:latest -<<EOF
            FROM busybox
            RUN echo "hello world"
            EOF
8. Build from a local build context, using a Dockerfile from stdin
    Use this syntax to build an image using files on your local filesystem, but using a Dockerfile from stdin. The syntax uses the -f (or --file) option to specify the Dockerfile to use, and it uses a hyphen (-) as filename to instruct Docker to read the Dockerfile from stdin:


        docker build [OPTIONS] -f- PATH
    The example below uses the current directory (.) as the build context, and builds an image using a Dockerfile that is passed through stdin using a here document.

            docker build -t myimage:latest -f- . <<EOF
            FROM busybox
            COPY somefile.txt ./
            RUN cat /somefile.txt
            EOF

9. Build from a remote build context, using a Dockerfile from stdin
  Use this syntax to build an image using files from a remote Git repository, using a Dockerfile from stdin. The syntax uses the -f (or --file) option to specify the Dockerfile to use, using a hyphen (-) as filename to instruct Docker to read the Dockerfile from stdin:

        docker build [OPTIONS] -f- PATH
        docker build -t myimage:latest -f- https://github.com/docker-library/hello-world.git <<EOF
        FROM busybox
        COPY hello.c ./
        EOF

10. Minimize the number of layers
    In older versions of Docker, it was important that you minimized the number of layers in your images to ensure they were performant. The following features were added to reduce this limitation:

    Only the instructions RUN, COPY, ADD create layers. Other instructions create temporary intermediate images, and don’t increase the size of the build.

    Where possible, use multi-stage builds, and only copy the artifacts you need into the final image. This allows you to include tools and debug information in your intermediate build stages without increasing the size of the final image.

11. Decouple applications
    Each container should have only one concern. Decoupling applications into multiple containers makes it easier to scale horizontally and reuse containers. For instance, a web application stack might consist of three separate containers, each with its own unique image, to manage the web application, database, and an in-memory cache in a decoupled manner.

    Limiting each container to one process is a good rule of thumb, but it’s not a hard and fast rule. 
    Use your best judgment to keep containers as clean and modular as possible. If containers depend on each other, you can use Docker container networks to ensure that these containers can communicate.

    Multistage build has been implemented in `docker-compose file` as a best practice of Dockerfile

### Learn how to install docker on your local machine and put the same containers running outside gitpod or codespaces.
Before pushing an image to docker hub, someone must have docker installed   I had some challenges installing docker on my local system, so I created an Ubuntu virtual machine using oracle virtual box. Then I installed docker, docker engine, docker engine cli, docker compose, containerd and docker desktop on it following this [official documentation from docker](https://docs.docker.com/desktop/install/ubuntu/).

### Scanning DockerFile in Github with Synx Open Source and and Github with Trivy 
- Install trivy via the commandline 

```
wget https://github.com/aquasecurity/trivy/releases/download/v0.20.0/trivy_0.20.0_Linux-64bit.deb && sudo dpkg -i trivy_0.20.0_Linux-64bit.deb

```

- Scan images 
```
trivy image <image-name>
```
- Sacn images and send vunerability results to a text file 

```
trivy image --format json --exit-code 1 $(docker-compose config --services)

This command does the following:
trivy image tells Trivy to scan Docker images.
--format json specifies that the output should be in JSON format.
--exit-code 1 tells Trivy to exit with a non-zero status code if any vulnerabilities are found.
$(docker-compose config --services) runs the docker-compose config command to get a list of all the services defined in your Docker Compose file, and passes them as arguments to the trivy command.
You can customize this command to scan specific services or images in your Docker Compose file. For example, to scan the backend-flask service in a Docker Compose file called docker-compose.yml, you could use the following command:

```
![vunerability_scanning](/assets/Capture_install_trivy_scan_33.PNG)


![vunerability_scanning](/assets/Capture_trivy_scan_report_34.PNG)

- Sign up to Synx free tier 
```
https://snyk.io/product
```
- Install synx on Gitpod 
```
npm install -g snyk 
cd ~/projects/my-project/ 
snyk monitor

```

![vunerability_scanning](/assets/Capture_scan_with_synx_35.PNG)

### Building Multi-Stage DockerFiles 
- Flask backend docker file 
```
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]

```

- Flask backend dockerfile written using the multi-stage build process 
```
FROM python:3.10-slim-buster AS base

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}

CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]


FROM python:3.10-slim-buster AS production

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install --no-cache-dir -r requirements.txt

COPY . .

ENV FLASK_ENV=production

EXPOSE ${PORT}

CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]


```
- Further explanation on what is happening above  below 
The above Dockerfile defines two stages: "base" and "production". In the "base" stage, the necessary dependencies and source code are copied and installed. Then, the development server is started.

In the "production" stage, the same dependencies and source code are copied and installed, but this time with the --no-cache-dir option to avoid caching pip packages. The Flask environment is set to "production", and the server is started.

The benefit of using multistage builds is that the final image is smaller because it only includes the necessary production dependencies and not the development dependencies, which are only needed during the build process. This can reduce the attack surface and make the image more secure. Additionally, by separating the build process into stages, the build time can be reduced, as only the necessary dependencies and files are copied between stages.

- Diagram illustrating the process 
```
            +----------------------------------------+
            |               Base Stage                |
            +----------------------------------------+
                        |           |
                        v           v
        +-------------------------+------------------+
        |  Copy dependencies and   |     Copy source   |
        |     install packages     |   code and files  |
        +-------------------------+------------------+
                        |           |
                        v           v
       +--------------------------+------------------+
       |  Start development server |   (no further    |
       |                           |    action)       |
       +--------------------------+------------------+
                                    |
                                    |
                                    v
            +----------------------------------------+
            |            Production Stage            |
            +----------------------------------------+
                        |           |
                        v           v
        +-------------------------+------------------+
        |  Copy dependencies and   |     Copy source   |
        |     install packages     |   code and files  |
        +-------------------------+------------------+
                        |           |
                        v           v
       +--------------------------+------------------+
       |  Start production server  |   (no further    |
       |                           |    action)       |
       +--------------------------+------------------+


```
