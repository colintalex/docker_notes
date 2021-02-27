Notes

Docker is TOOL for running applications in an isolated environment
Similar to VirtualMachine
App run in same environment


Containers vs Virtual Machines (VM's)

Container are an abstraction at the app layer that packages code and depedencies together. Multiple containers can run on the same machine.

VM's are an abstraction of the physical hardware turning one server into many servers.

## Benefits of Docker
- Run container in seconds instead of mins
- Less resources results in less disj space
- Less mem needed
- Not need a full os
- Deployment
- Testing, can test locally

## Docker Images
- Image is a template for creating an environment of your choice
- Snapshot, allows you to rollback if errored
- Has everything you need to run your apps
- Contains **OS, Software, and App Code**
- hub.docker.com has a list of all popular and **OFFICIAL** images (NGINX, mongoDB, alpine, node, redeix, ubuntu, etc)


## Docker Container
- Running instance of an image, simple as that. 

### Example
```
$ docker images
->REPOSITORY    TAG     IMAGE ID         CREATED     SIZE
->nginx         latest  98asd9ad8as      4 days ago  103mb
```

To run a container(instance of the image):

```
docker run nginx:latest
```

This will start up a container in the background. 
To view the container stats, run
```
docker container ls
```

You will see something like

```
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
92a40192e538   nginx:latest   "/docker-entrypoint.…"   50 seconds ago   Up 49 seconds   80/tcp    trusting_mccarthy
```

If you close the tab running the container, the list will be empty.


To run a container in detached mode
```docker run -d nginx:latest```


Exposing Ports

In the Nginx container, the Port is 80, as seen above. ```80/tcp```


On your host, you will have a localhost address. To maop the localhost to the proper port, say using localhost:8080 and Port 80 for container, use ```-p 8080:80```


## Stopping a Container

To stop a container, run
```
docker stop *container_id*
```


## Test out the port exposure

Run 
```
docker run -d -p 8080:80 nginx:latest
```

Now go to localhost:8080 on your browser, and you should see NGINX page, verifying succesful connection.


## Exposing Multiple Ports
 
 You can also connect localhost:3000 and localhost:8080 to mapped to Port 80
You are essentially connecting ports from your host to your Container ports.

 ```
docker run -d -p 8080:80 ip 3000:80 nginx:latest
 ```

 Why would you want to do this?
 - TBD


## Deleting a Container

```
docker rm *container name or id*
```
Will return the Container ID but it is no longer listed when running ```docker ps -a```

**You cannot remove a running container**, unless you add the `-f` tag.

## Adding a name to the Container

```
docker run --name **yourcustomname** -d -p 3000 ....
```

Now it's easier to manage!

```
docker stop *yourcustomname*
```
```
docker start *yourcustomname*
```

# Volumes
#### Docker volumes allows the sharing of data, using files and folders.
- Shares between host and containers
- Shares between other containers

## Volumes Host & Container
- Say you have a file on your host machine, that file will also appear inside the container, inside of the volume. 
- By adding a file inside of the container volume, it will also appear inside the host machine. 


### Creating a Volume between Host and Container

Example file -> index.html
Example Path for Host File -> Users/colinalex/Desktop/website
Example Path for Container File -> /usr/share/nginx/html


- In a code editor create basic HTML file, where you save it will be the path for your Host file.

- In your terminal, navigate into the folder. Then run:
```
docker run --name website -v $(pwd):/usr/share/nginx/html:ro -d -p 8080:80 nginx
```

The `$(pwd)` indicates the present working directory, you can always enter the file path. 
The `/usr/share/nginx/html` is the correct path.

We just effectively mounted the folder we navigated to in the volume. 
When you vist port 8080, you will see the Index.html file.

### Accessing the Container
```
docker exec -it website bash
```
*Similar to SSH access on a server.

You can create a file in the container's bash environment if allowed(write access), and the new file will be replicated on the host. Pretty sweet!

## Volumes between Containers

Shares the contents of a folder or file between multiple containers.

```
docker run --name website-copy --volumes-from website -d -p 8081:80 nginx
```

We specified port `8081:80` instead of `8080:80` because we still have a container running on port `8000:80`

You now have two different containers running on the host, with access to the same volumes.

## The Dockerfile
- So far, we've only used the pre-built, official Nginx image
- The Dockerfile let's you build your own images(container blueprints).

- The Dockerfile is a series of steps that determines how your image is built. 

*See Dockerfile reference in the Docs.*

 ### Creating a Dockerfile and Creating an Image from Dockerfile
- So far, we've been mounting a volume. (shared folder access from host to container). This is good for development, but to create a custom image you don't need to mound a volume. 


Quick, a check to see if we have a volume and a container.
```
docker image ls
```
```
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
nginx        latest    35c43ace9216   10 days ago   133MB
```

 ```
 docker ps --format=$FORMAT

 ->CONTAINER ID   IMAGE          COMMAND                  CREATED             STATUS             PORTS     NAMES
91b78cc9179f   nginx:latest   "/docker-entrypoint.…"   About an hour ago   Up About an hour   80/tcp    goofy_visvesvaraya
 ```

Remember, the Dockerfile contains everything your app needs to run. 

- Create a file named `Dockerfile`

`Dockerfile`
```
FROM nginx:latest
ADD ./usr/share/nginx/html <-- Adds everything from current dir to the * nginx/html directory
```

Congrats! You've created a basic Dockerfile that can create an image!


## Creating an Image from Dockerfile

```
docker build --tag {name:tag} {where your Dockerfile is located}
```

Example
```
docker --build --tag website:latest .
```

Press Enter and you should see successful confirmation for each step. Running `docker ls` should show your new image. 

**Remember this is an image, which can be instanced as a container.**

<br>
<br>

### Now let's create a container from the image!
```
docker run --name website -p 8080:80 -d website:latest
```
