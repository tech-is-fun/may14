# Docker and Ngnix

> Set up docker-compose, docker to run nginx and serve the sample web site. 

## Workshop Instructions
Read over the description of how to create the Docker files and then see below for instructions on how to launch the app.


## The docker files
You've set up and built the sample web client as described previously [here](./client.md)

The nginx docker file is very simple, for now. It has one line that loads the nginx image from the docker hub repository.
In fact we could do without the docker file and place this image line in the docker compose file but we'll need this
nginx docker file for the next stage to set up the proxy to the API.

nginx/DOCKERFILE
```
FROM nginx
``` 

The docker-compose file is more interesting. It links our static web site to the nginx.


./stage2-docker-compose.yml
```
version: "3"
services:
  nginx:
    container_name: nginx-stage2
    build:
      context: ./nginx
      dockerfile: Dockerfile
    ports:
      - "8083:80"
    volumes:
      - "./client/my-project/dist/:/usr/share/nginx/html/"
```

Note the port mapping sets us up for development. On your development box open a browser and navigate to
http://localhost:8083.  What you see is the same as what will be served from the production server. 

We need make some overrides for production (otherwsie we run the exact same configuration on production).
First expose port 80 for our production server and set the flag to restrt this service after a reboot.

./stage2-prod-docker-compose.yml
```
version: "3"
services:
  nginx:
    ports:
      - "80:80"
    restart: "always"      
```

## Launch the app on dev

To run production on your development box: 
``` 
docker-compose -f stage2-docker-compose.yml up --build
```
See your app at http://localhost:8083

### Mac setup
Did you see an error like the following on the Mac development environment?

> ERROR: for nginx-stage2  Cannot start service nginx: b'Mounts denied: 
The path ... is not shared from OS X
and is not known to Docker. You can configure shared paths from Docker -> Preferences... -> File Sharing.
See https://docs.docker.com/docker-for-mac/osxfs/#namespaces for more info.'

As it says you need to open the Docker preferences and add in the path to your web site static files. 

## Launch the app on prod

Use ssh to return to your production server and return to your project folder.  Be sure you've checked out
the stage2 branch.  
``` 
# on production
git checkout -b stage2
git status
# check to see that you see the stage2 branch

# Launch the app with production overrides
sudo docker-compose -f stage2-docker-compose.yml -f stage2-prod-docker-compose.yml up --build
```

After a moment you will see 
``` 
Creating nginx-stage2 ... done
Attaching to nginx-stage2
```
This is when you can open a browser and see your app at http://\<your server ip address>

This is end of stage 2.  