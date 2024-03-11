# Output from github.com action
```
Run whoami
nsingleton
CODE_OF_CONDUCT.md
CONTRIBUTING.md
Dockerfile
Gruntfile.js
LICENSE
Procfile
README.md
app
app.json
artifacts
config
cypress.json
docker-compose.yml
nodemon.json
package-lock.json
package.json
server.js
test
#0 building with "default" instance using docker driver

#1 [web internal] load build definition from Dockerfile
#1 transferring dockerfile: 633B done
#1 DONE 0.0s

#2 [web internal] load metadata for docker.io/library/node:12-alpine
#2 DONE 0.3s

#3 [web internal] load .dockerignore
#3 transferring context: 108B done
#3 DONE 0.0s

#4 [web stage-0 1/4] FROM docker.io/library/node:12-alpine@sha256:d4b15b3d48f42059a15bd659be60afe21762aae9d6cbea6f124440895c27db68
#4 DONE 0.0s

#5 [web internal] load build context
#5 transferring context: 2.98MB 0.1s done
#5 DONE 0.1s

#6 [web stage-0 2/4] WORKDIR /usr/src/app/
#6 CACHED

#7 [web stage-0 3/4] COPY package*.json /usr/src/app/
#7 CACHED

#8 [web stage-0 4/4] RUN npm install --production --no-cache
#8 CACHED

#9 [web stage-1 3/5] COPY --from=0 /usr/src/app/node_modules node_modules
#9 CACHED

#10 [web stage-1 2/5] WORKDIR /home/node/app
#10 CACHED

#11 [web stage-1 4/5] RUN chown node:node /home/node/app
#11 CACHED

#12 [web stage-1 5/5] COPY --chown=node . /home/node/app
#12 DONE 0.1s

#13 [web] exporting to image
#13 exporting layers 0.1s done
#13 writing image sha256:736219588b08a7654d7a9d9057407a079fd106e90f47f24952dd179e957393a0 done
#13 naming to docker.io/library/itas276-nodegoat-nsp-web done
#13 DONE 0.1s
Network itas276-nodegoat-nsp_default Creating
Network itas276-nodegoat-nsp_default Created
Container itas276-nodegoat-nsp-web-1 Creating
Container itas276-nodegoat-nsp-mongo-1 Creating
Container itas276-nodegoat-nsp-web-1 Created
Container itas276-nodegoat-nsp-mongo-1 Created
Container itas276-nodegoat-nsp-mongo-1 Starting
Container itas276-nodegoat-nsp-web-1 Starting
Container itas276-nodegoat-nsp-web-1 Started
Container itas276-nodegoat-nsp-mongo-1 Started
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
aadeb029e55e itas276-nodegoat-nsp-web "docker-entrypoint.s…" 1 second ago Up Less than a second 41910/tcp, 0.0.0.0:41910->4000/tcp, :::41910->4000/tcp itas276-nodegoat-nsp-web-1
c72120325144 mongo:4.4 "docker-entrypoint.s…" 1 second ago Up Less than a second 27017/tcp itas276-nodegoat-nsp-mongo-1
7f76db085c6c itas276-nodegoat-sb-web "docker-entrypoint.s…" 21 minutes ago Up 21 minutes 40103/tcp, 0.0.0.0:40103->4000/tcp, :::40103->4000/tcp itas276-nodegoat-sb-web-1
fc47028ea91d mongo:4.4 "docker-entrypoint.s…" 21 minutes ago Up 21 minutes 27017/tcp itas276-nodegoat-sb-mongo-1
c04e64c8d9a9 itas276-assignment1-sg-web "docker-entrypoint.s…" 17 hours ago Up 17 hours 41011/tcp, 0.0.0.0:41011->4000/tcp, :::41011->4000/tcp itas276-assignment1-sg-web-1
b22cdb9553c4 mongo:4.4 "docker-entrypoint.s…" 18 hours ago Up 18 hours 27017/tcp itas276-assignment1-sg-mongo-1
```

# Output from self hosted runner

![Screenshot of terminal](https://cdn.discordapp.com/attachments/1055979588553031822/1216818954694033428/Screenshot_2024-03-11_112614.png?ex=6601c5f6&is=65ef50f6&hm=8001ad6e6eb1602fd1eac2fb849fdadc04160bfb418e8e4e003ead119770fed5&)



### Workflow Action File

```yaml
name: Build and Push to Docker Hub

# Trigger the workflow on push or pull request events on the master branch
on:
  push:
    branches: [ "master" ]
  pull_request:
    branches: [ "master" ]

jobs:
  deploy:
    name: "Deploy to ITAS WMD server"
    runs-on: self-hosted  # Use a self-hosted runner for deployment

    steps:
      - name: Copy source code files
        uses: actions/checkout@master  # Checkout the source code

      - name: Test files are visible
        run: |
          pwd  # Print the current working directory
          whoami  # Print the current user
          ls  # List the files in the current directory
          docker-compose up -d  # Start the Docker Compose services in detached mode
          docker ps  # List the running Docker containers

  build-and-push:
    runs-on: ubuntu-latest  # Use the Ubuntu environment for building and pushing the Docker image

    steps:
      - uses: actions/checkout@v3  # Checkout the source code

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1  # Set up Docker Buildx for building the image

      - name: Login to Docker Hub
        uses: docker/login-action@v1  # Log in to Docker Hub
        with:
          username: ${{ secrets.DOCKER_USERNAME }}  # Use the Docker Hub username stored as a secret
          password: ${{ secrets.DOCKER_PASSWORD }}  # Use the Docker Hub password stored as a secret

      - name: Build and push
        uses: docker/build-push-action@v2  # Build and push the Docker image
        with:
          context: .  # Use the current directory as the build context
          file: ./Dockerfile  # Specify the path to the Dockerfile
          push: true  # Push the built image to Docker Hub
          tags: apex6599/node-goat:latest  # Tag the image with the specified tag (replace with your Docker Hub username and image repository)
```

## Dockerfile

```dockerfile
FROM node:12-alpine
ENV WORKDIR /usr/src/app/
WORKDIR $WORKDIR
COPY package*.json $WORKDIR
RUN npm install --production --no-cache

FROM node:12-alpine
ENV USER node
ENV WORKDIR /home/$USER/app
WORKDIR $WORKDIR
COPY --from=0 /usr/src/app/node_modules node_modules
RUN chown $USER:$USER $WORKDIR
COPY --chown=node .github/workflows $WORKDIR
# In production environment uncomment the next line
#RUN chown -R $USER:$USER /home/$USER && chmod -R g-s,o-rx /home/$USER && chmod -R o-wrx $WORKDIR
# Then all further actions including running the containers should be done under non-root user.
USER $USER
EXPOSE 41910
```

## DockerCompose

```yaml
version: "3.7"

services:
  web:
    build: .
    environment:
      NODE_ENV:
      MONGODB_URI: mongodb://mongo:27017/nodegoat
    command: sh -c "until nc -z -w 2 mongo 27017 && echo 'mongo is ready for connections' && node artifacts/db-reset.js && npm start; do sleep 2; done"
    ports:
      - "41910:4000"

  mongo:
    image: mongo:4.4
    user: mongodb
    expose:
      - 27017
```

## Synopsis and lessons learned 

In this DevOps class project, we successfully deployed a self-hosted runner and enhanced our Dockerfile to expose a specific port, enabling secure access to the Docker image we built and deployed to our server.

The process began by setting up a self-hosted runner, which allowed us to have full control over the environment and resources used for our deployment pipeline. By configuring the runner on our own infrastructure, we ensured that our specific requirements and dependencies were met.

Next, we focused on updating our Dockerfile to expose the necessary port. This step was crucial to enable external access to the containerized application running within the Docker image. By specifying the port in the Dockerfile, we established a communication channel between the container and the host system.

With the Dockerfile updated and the port exposed, we proceeded to build the Docker image using the defined configuration. This image encapsulated our application along with its dependencies, ensuring a consistent and reproducible runtime environment.

Once the image was built, we deployed it to our server, leveraging the power of containerization. By running the container based on the built image, we achieved a seamless deployment process, eliminating the need for manual setup and configuration on the server.

To securely access the deployed Docker image, we utilized SSH tunneling. By establishing an SSH tunnel, we created an encrypted connection between our local machine and the server hosting the container. This secure tunnel allowed us to interact with the containerized application as if it were running locally, while ensuring the confidentiality and integrity of the transmitted data.

Through this project, we gained hands-on experience in deploying a self-hosted runner, updating a Dockerfile to expose ports, building and deploying Docker images, and securely accessing the containerized application using SSH tunneling. These skills are essential in modern DevOps practices, enabling efficient and secure deployment workflows.

