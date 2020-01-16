# Docker notes

## Node

```
FROM node:7.7.2-alpine
WORKDIR /usr/app
COPY package.json .                  # copy individual file to working directory in container
COPY node_modules ./node_modules     # copy folder and contents to the same location in container
RUN npm install --quiet
COPY . .
```

## .NET Core

You need to change the default listing URL: `.UseUrls("http://*:5000")`

```
# Copy everything and build on build image
FROM microsoft/aspnetcore-build:2.0 AS build-env
COPY . ./
RUN dotnet publish -c Release -o out
RUN ls -a

# Build runtime image
FROM microsoft/aspnetcore:2.0
WORKDIR /Xero.Api.Example.MVC
COPY --from=build-env /Xero.Api.Example.MVC/out .
RUN ls -a
ENTRYPOINT ["dotnet", "./Xero.Api.Example.MVC.dll"]
```

### docker-compose.yml

```
version: '2'
services:
  web:
    build: .
    command: npm run dev
    volumes:
      - .:/usr/app/ 
      - /usr/app/node_modules 
    ports:
      - "3000:3000" 
    depends_on: 
      - postgres 
    environment: 
      DATABASE_URL: postgres://todoapp@postgres/todos 
  postgres: 
    image: postgres:9.6.2-alpine 
    environment: 
      POSTGRES_USER: todoapp
      POSTGRES_DB: todos
```

# ECR

These commands will"

1. AWS Cli Login
2. build an image and tag it with philtest
3. create a new tag that referes to the old tag
4. push image with new tag to ECR

```
aws ecr get-login --no-include-email --region us-west-1
docker build -t philtest .
docker tag philtest:latest 111111111111.dkr.ecr.us-west-1.amazonaws.com/philtest:latest
docker push 111111111111.dkr.ecr.us-west-1.amazonaws.com/philtest:latest
```

# Useful commands

| Command | Does | Options |
| --- | --- | --- |
| `docker build` | Builds a docker image from Dockerfile and puts into your local image repo | 	-t <name> => tags that image with a name |
| `docker images`	| lists images saved in your local image repo | |
| `docker rmi <image>` |	removes single image	| |
| `docker rm $(docker kill $(docker ps -aq))` | Kill all containers and removes them	| |
| `docker ps` | Shows running containers	Also use docker-compose ps
| `docker run -t -p 8000:8000 image:version` | Run the specified image on the terminal	| |
| `docker run -d -p 8000:8000 image:version` | Run the specified image in the background | |
| `docker-compose <up/down>` | will run your compose file and keep the terminal open	-d => run as background process |
| `docker-compose logs` | will show the logs if running in background | |
| `docker-compose ps` | shows active containers | -a => shows all active/inactive |
| `docker run -t -i image:version /bin/sh` | Access the shell on the container (e.g. to view the file system) | |
| `docker system prune --volumes -f `| Removes all dangling images and those that are not used	| |
| `docker ps -q xargs docker stop ; docker system prune -a` | On a mac, stop and remove all	| |
| `docker exec -it 40e725064583 sh` | open bash on container	| |
|`docker rmi $(docker images -q)`| Delete all images | |
|`docker rm $(docker ps -a -q)` | Delete all containers | |
