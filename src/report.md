# Simple Docker Report

## Part 1. Ready-made docker

As the final goal of your little practice you have immediately chosen to write a docker image for your own web server, so first you need to deal with a ready-made docker image for the server.
You chose a pretty simple **nginx**.

**== Task ==**

### Take the official docker image from **nginx** and download it using `docker pull`.
![1-1](/misc/images/part1/1-1.png)
### Check for the docker image with `docker images`
![1-2](/misc/images/part1/1-2.png)
### Run docker image with `docker run -d [image_id|repository]`
![1-3](/misc/images/part1/1-3.png)
### Check that the image is running with `docker ps`
![1-4](/misc/images/part1/1-4.png)
### View container information with `docker inspect [container_id|container_name]`
![1-5](/misc/images/part1/1-5.png)
### From the command output define and write in the report the container size, list of mapped ports and container **ip**

| SIZE  | PORTS  |     IP     |
| :---: | :----: | :--------: |
| 142MB | 80/tcp | 172.17.0.3 |


### Stop docker image with `docker stop [container_id | container_name]`
![1-6](/misc/images/part1/1-6.png)
### Check that the image has stopped with `docker ps`
![1-6](/misc/images/part1/1-7.png)
### Run docker with mapped ports 80 and 443 on the local machine with *run* command
![1-6](/misc/images/part1/1-8.png)
### Check that the **nginx** start page is available in the browser at *localhost:80*
![1-6](/misc/images/part1/1-9.png)
### Restart docker container with `docker restart [container_id|container_name]`
### Check in any way that the container is running
![1-6](/misc/images/part1/1-10.png)

## Part 2. Operations with container

Docker image and container are ready. Now we can look into **nginx** configuration and display page status.

**== Task ==**

### Read the *nginx.conf* configuration file inside the docker container with the *exec* command
![2-0](/misc/images/part2/2-0.png)
### Create a *nginx.conf* file on a local machine
`sudo mkdir /etc/nginx; sudo touch /etc/nginx/nginx.conf`
### Configure it on the */status* path to return the **nginx** server status page
![2-1](/misc/images/part2/2-1.png)
### Copy the created *nginx.conf* file inside the docker image using the `docker cp` command
![2-2](/misc/images/part2/2-2.png)
### Restart **nginx** inside the docker image with *exec*
![2-3](/misc/images/part2/2-3.png)
### Check that *localhost:80/status* returns the **nginx** server status page
![2-4](/misc/images/part2/2-4.png)
### Export the container to a *container.tar* file with the *export* command
`docker export 699a3f94151e > container.tar`
### ![2-5](/misc/images/part2/2-5.png)
### Stop the container
![2-6](/misc/images/part2/2-6.png)
### Delete the image with `docker rmi [image_id|repository]`without removing the container first
![2-7](/misc/images/part2/2-7.png)
### Delete stopped container
![2-8](/misc/images/part2/2-8.png)
### Import the container back using the *import*command
![2-9](/misc/images/part2/2-9.png)
### Run the imported container
![2-10](/misc/images/part2/2-10.png)
### Check that *localhost:80/status* returns the **nginx** server status page
![2-11](/misc/images/part2/2-11.png)

## Part 3. Mini web server

It's time to take a little break from the docker to prepare for the last stage. It's time to write your own server.

**== Task ==**

### Write a mini server in **C** and **FastCgi** that will return a simple page saying `Hello World!`
`sudo apt install libfcgi-dev` - install library FastCgi.
### Run the written mini server via *spawn-fcgi* on port 8080
`sudo apt install spawn-fcgi` - install spawn-fcgi service
![3-0](/misc/images/part3/3-0.png)
### Write your own *nginx.conf* that will proxy all requests from port 81 to *127.0.0.1:8080*
`sudo apt install nginx` - install Nginx
### ***nginx.conf:***
`events {}`

`http {`
`	server  {`
`		listen 81;`
`		server_name fcgiserver;`
`		location / {`
`			fastcgi_pass 127.0.0.1:8080;`
`		}`
`	}`
`}`

### Check that browser on *localhost:81* returns the page you wrote
![3-2](/misc/images/part3/3-2.png)
### Put the *nginx.conf* file under *./nginx/nginx.conf* (you will need this later)

## Part 4. Your own docker

Now everything is ready. You can start writing the docker image for the created server.

**== Task ==**

*When writing a docker image avoid multiple calls of RUN instructions*

### Write your own docker image that:
### 1) builds mini server sources on FastCgi from [Part 3](#part-3-mini- web-server)
### 2) runs it on port 8080
### 3) copies inside the image written *./nginx/nginx.conf*
### 4) runs **nginx**.
_**nginx** can be installed inside the docker itself, or you can use a ready-made image with **nginx** as base._
### Build the written docker image with `docker build`, specifying the name and tag
`docker build --no-cache -t mydock:new .`
### Check with `docker images` that everything is built correctly
`docker images`
### Run the built docker image by mapping port 81 to 80 on the local machine and mapping the *./nginx* folder inside the container to the address where the **nginx** configuration files are located (see [Part 2](#part-2-operations-with-container))
`docker run -p 80:81 -v "$PWD/nginx/":/etc/nginx mydock`
### Check that the page of the written mini server is available on localhost:80
![h-w](/misc/images/part6/h-w.png)
### Add proxying of */status* page in *./nginx/nginx.conf* to return the **nginx** server status
		location /status {
			stub_status on;
			access_log off;
		}
### Restart docker image
*If everything is done correctly, after saving the file and restarting the container, the configuration file inside the docker image should update itself without any extra steps

![4-1](/misc/images/part4/4-1.png)
### Check that *localhost:80/status* now returns a page with **nginx** status
![4-2](/misc/images/part4/4-2.png)

## Part 5. **Dockle**

Once you've written the image, it's never a bad idea to check it for security.

**== Task ==**

### Check the image from the previous task with `dockle [image_id|repository]`
***CIS-DI-0001: Create a user for the container***

***DKL-DI-0006: Avoid latest tag***

***DKL-DI-0005: Clear apt-get caches***
### Fix the image so that there are no errors or warnings when checking with **dockle**

1) CIS-DI-0001: `useradd -g daemon -d /home/dockle -m -s /bin/bash dockle`
`USER dockle`
2) DKL-DI-0005: `RUN apt-get clean && rm -rf /var/lib/apt/lists/*`
3) /var/run/nginx.pid failed (13 permission denied): `RUN chown -R dockle:daemon /var/cache/nginx/ /var/run/ /var/log/nginx/ `


## Part 6. Basic **Docker Compose**

There, you've finished your warm-up. Wait a minute though...
Why not try experimenting with deploying a project consisting of several docker images at once?

**== Task ==**

### Write a *docker-compose.yml* file, using which:
### 1) Start the docker container from [Part 5](#part-5-dockle) _(it must work on local network, i.e., you don't need to use **EXPOSE** instruction and map ports to local machine)_
### 2) Start the docker container with **nginx** which will proxy all requests from port 8080 to port 81 of the first container
![6-0](/misc/images/part6/6-0.png)
### Map port 8080 of the second container to port 80 of the local machine
![6-1](/misc/images/part6/6-1.png)
### Stop all running containers
`docker stop $(docker ps -q)`
### Build and run the project with the `docker compose build` and `docker compose up` commands
### Check that the browser returns the page you wrote on *localhost:80* as before
![h-w](/misc/images/part6/h-w.png)