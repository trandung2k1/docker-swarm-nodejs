# Docker-swarm-nodejs
A sample of Docker Swarm with nodejs

Create a new folder called api.
```bash
$ cd api
```

Create a new file index.js and paste the code below : 

```node
const http = require('http');
const os = require('os');

http.createServer(function (req, res) {
    res.writeHead(200, {'Content-Type': 'text/html'});
    res.end(`<h1>I'm ${os.hostname()}</h1>`);
}).listen(8080);
```

#### Create Dockerfile
Now we need to dockerize the app, we’ll create a file named Dockerfile with the following code:

```docker
FROM node
RUN mkdir -p /usr/src/app
COPY index.js /usr/src/app
EXPOSE 8080
CMD [ "node", "/usr/src/app/index" ]
```
#### Build the Docker image
To build a docker image of our newly awesome Node.js app from the docker file instructions; we’ll write the docker build command line, where our Dockerfile file is located:

```bash
$ docker build -t api .
```

#### Let's run
Now we have a docker image of our simple (and awesome) Node.js app, and we can create containers from that image.

Let's say we want 3 running containers of that image and all behind a load balancing server.
For our HTTP server we’ll use HAProxy that will listen to port 3000
Let’s create a docker-compose.yml file:

```docker
version: '3'

services:
    api:
        image: api
        ports:
            - 8080
        environment:
            - SERVICE_PORTS=8080
        deploy:
            replicas: 3
            update_config:
                parallelism: 5
                delay: 10s
            restart_policy:
                condition: on-failure
                max_attempts: 3
                window: 120s
        networks:
            - trandung

    proxy:
        image: dockercloud/haproxy
        depends_on:
            - api
        environment:
            - BALANCE=leastconn
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
        ports:
            - 3000:80
        networks:
            - trandung
        deploy:
            placement:
                constraints: [node.role == manager]

networks:
    trandung:
        driver: overlay

 ```
 
 
 #### Docker SWARM !!!!!
 Now let’s create a swarm (with one computer for now, but you can easily add more to the swarm). To do this we'll write docker swarm init and we created a swarm!! 
 
 ```bash
 docker swarm init
 ```
 
 It’s also added our current computer to the swarm, and since our computer is the first it’s also the manager of the swarm.
 
 Let's build the stack with the following comand line :
 
 ```bash
 docker stack deploy --compose-file=docker-compose.yml production
 ```
 We are doing two things : 
  1 - Build the services 
  2 - deploy them to our local stack called production
  
 Once done you browse http://localhost:3000 and see the nodejs message, 
 
 Hitting F5 will display a different hostname since HAproxy will load balance the request.
 
 