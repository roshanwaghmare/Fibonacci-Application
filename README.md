# Multi Container Application Fibonacci-Application

Section : 08
We are creating application that calculate Fibonacci value when we enter index numbers eg: 1,2,3,4 etc 

![Screenshot 2024-01-29 133711](https://github.com/roshanwaghmare/Fibonacci-Application/assets/142305817/b164bdf6-d39d-440e-93ea-ef75650c9ec1)

## Steps


## to open file and folder directly form terminal u need to use below cmd

```
sudo apt install nautilus
xdg-open .
open .
```
this setup is not working for us so we find below solution 

1 now we have to get all the java code from our local machine to ubuntu terminal so use below

````
Open .

was not working in my ubuntu so use below path
cd .. till main directory
cd mnt/
i ahev copied complex folder from downloads and paste in the c dirve 
cp -r complex/ /home/roshanw/
````
## hopefully you now have a complex directory and inside there is the client server in worker folders  each of those folders represent the React server the express API and the worker process as well. We're now not gonna start the process of adding Docker containers to each of these applications.

We can start each of them up inside of a development environment. We're gonna take the React project, the express API and the worker as well and we're going to make Dev Docker files for each one.

## now will create three dockerfile for each React server the express API and the worker { Dev Environment}

at the project files inside of each of those directories and inside of each of them we're gonna set up a pretty similar Docker file workflow. Remember that each one of these projects is probably gonna have a package.json file that records all of the dependencies of our project. We're going to copy over that package.json file as step number one. We're then going to run a NPM install and then we'll copy over everything else. The last thing to keep in mind is that we're gonna set up a Docker composed file and that Docker compose is gonna set up volumes for each of these projects so that we kind of share all of the source code
**inside of each project and that's what's gonna make sure that we don't have to rebuild our image entirely from scratch every time that we make one tiny little change.**



----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

now will create **dockerfile.dev** in **client / Server /Worker** folders 
docker build -f Dockerfile.dev {**.**} this means build in current working directory 
````
Client

FROM node:16-alpine
WORKDIR '/app'
COPY ./package.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "start"]

````
workwer and server start cmd will be diffrent

**the startup command is gonna be just a little bit different for the server and the worker. If you open up the package.json file, you'll notice that we set up a dev script in both of these of simply nodemon. Nodemon is a little command line tool that can be used to automatically reload your entire project whenever any of the source code inside of your project is changed. And so we're gonna take advantage of that as we are running our docker containers to make sure that any time we set up a volume and our source code changes, and the volume updates as well, we'll get our application to automatically restart with the nodemon tool.**
````
FROM node:14.14.0-alpine
WORKDIR '/app'
COPY ./package.json ./
RUN npm install
COPY . .
CMD ["npm", "run", "dev"]

````
## Docker compose

## Docker Compose file, is to make sure that it's a lot easier to start up each of these different images as containers with the appropriate arguments. For example, we need to make sure that the Express server is available on some given port.

will create docker compose in complex directory 

## development environment, we will have a complete copy of the Express server, a complete copy of Redis, and a complete copy of Postgres

## Once we get these three pieces put together we'll then come back and start to add in the React application, the worker, and the Nginx server as well.

**{We probably need to set up some volumes
to make sure that anytime we change some of our source code,
the container source code updates as well.}**

docker-compose.yml
````
version: '3'
services:
  postgres:
    image: 'postgres:latest'
  redis:  
    image: 'redis:latest'
  server:
    build:
      dockerfile: dockerfile.dev
      context: ./server
    **volumes:
      - /app/node_modules
      - ./server:/app**

**Inside the container, don't try to override this folder. Don't try to override it, don't try to redirect access to it, just leave that folder as is. And then in addition to that, we're gonna say, look at the server directory, and copy everything inside there into the app folder of the container**
````
## Environment Variable

## The first step is when we build the image, that's the kind of preparation part. That's where we create a new image. And then at some point in the future when we actually run the container, that's the second part, that's when we actually take the image and create an instance of the container out of it. When we set up an environment variable inside of a Docker Compose file, we are setting up an environment variable that is applied at run time. So only when the container is started up.

````
  environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - PGUSER=postgres
      - PGHOST=postgres
      - PGDATABASE=postgres
      - PGPASSWORD=postgres_password
      - PGPORT=5432
Use documents on docker hub for name and port info it works as key and value 
````
now we have also added client and worker sever as well in docker-compose

````
 client:
    build: 
      dockerfile: Dockerfile.dev
      context: ./client
    volumes:
      - /app/node_modules
      - ./client:/app
  worker:
    build: 
      dockerfile: Dockerfile.dev
      context: ./worker
    volumes:
      - /app/node_modules
      - ./worker  
````
## We have not yet set up any port mapping whatsoever. So there's no port mapping to expose the server to the outside world. And there's no mapping for the React project

![as](https://github.com/roshanwaghmare/Fibonacci-Application/assets/142305817/36b15bf8-9991-4e92-91ec-27659d9ff827)

## Routing with Nginx

now we have not maped any ports until 

## so first we will create on **defualt.conf** file 

Nginx is going to watch for requests from the outside world and then route them to appropriate servers. These servers are kind of behind Nginx, you cannot access these servers unless you go through the Nginx server. And so Nginx refers to these as upstream servers,

Create React App by default port 3000, our server by default port 5,000. Now, the client right there, client:3000 and server:5000, those are both actual addresses. And so client and server are actual host names or essentially URLs that Nginx is going to try to direct traffic to. We are specifically using client and server right there because those are the names of the client and server services we put together inside of our Docker composed file.

After we put together those two quick definitions we'll then tell Nginx that it's going to listen by default on port 80. Now, remember this is port 80 inside the container and so it doesn't really matter if we say port 80 or we say port 5 billion, at the end of the day when we run our Docker composed file we can change the actual port that is more or less exposed to the outside world,

 "Essentially, if anyone comes to the slash

or kind of route route,

we wanna send them to the client upstream,"

so that's this upstream right here,

"... and if anyone goes to /API,

send them to the server upstream."


------------------------------------------------------------------------------------------------------------------------------------
Now will create nginx folder inside our complex and add **deafult.conf **file and **dockerfile.dev** adn will also make chnages to** docker compose**

````
default.conf

upstream client {
    server client:3000;
}

upstream api {
    server api:5000;
}

server {
    listen 80;

    location / {
        proxy_pass http://client;
    }

    location /{
        rewrite /api/(.*) /$1 break;
        proxy_pass http://api;
    }
}
````

```
dockerfile.dev
FROM nginx
COPY ./default.conf /etc/nginx/conf.d/default.conf

````
{{important note here we have chnaged **sever** name to **api** for our understanding this is express server for api calls}}

will also need to add nginx in docker compose

```
nginx:
    restart: always
    build:
      dockerfile: Dockerfile.dev
      context: ./nginx
    ports: 
      - '3050:80' 
```

