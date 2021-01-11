# local Docker

# Host requirements

## DNS

This known issue is discussed in detail here: https://development.robinwinslow.uk/2016/06/23/fix-docker-networking-dns/

Simple fix is to create the file `/etc/docker/daemon.json` and insert:

```
{
    "dns": ["10.0.0.2", "8.8.8.8"]
}
```

where `10.0.0.2` is the first DNS server your machine requests records from, and `8.8.8.8` is the fallback DNS server, google in this case.

## IP Forwarding

In order for containers to communicate with the outside world to install packages, etc, forwarding will need to be enabled. Simple fix here: https://linuxconfig.org/how-to-turn-on-off-ip-forwarding-in-linux

# Dockerfile

In terms of size, images are about:

* node 650MB
* -slim 250MB
* alpine 5MB

Help

* [Image source](https://github.com/nodejs/docker-node/tree/master/10)
* Derick Baily on [selecting node.js images](https://derickbailey.com/2017/03/09/selecting-a-node-js-image-for-docker/)
* [Building efficient Dockerfiles](http://bitjudo.com/blog/2014/03/13/building-efficient-dockerfiles-node-dot-js/)
* [Lessons from building node apps in Docker](http://jdlm.info/articles/2016/03/06/lessons-building-node-app-docker.html)

# Docker files

https://github.com/jessfraz/dockerfiles

## That I've built

* BinaryMistSite [Dockerfile](https://gitlab.com/binarymist/BinaryMistSite/blob/master/Dockerfile)
* NodeGoat [Dockerfile](https://github.com/binarymist/NodeGoat/blob/master/Dockerfile)

### purpleteam-orchestrator
 
* [Dockerfile](https://github.com/purpleteam-labs/purpleteam-orchestrator/blob/main/Dockerfile)
* [compose files](https://github.com/purpleteam-labs/purpleteam-orchestrator/tree/main/compose)

### purpleteam-app-scanner

* [Dockerfile](https://github.com/purpleteam-labs/purpleteam-app-scanner/blob/main/Dockerfile)

### purpleteam-s2-containers

* app-slave [docker-compose](https://github.com/purpleteam-labs/purpleteam-s2-containers/blob/main/app-slave/docker-compose.yml)
* selenium-standalone [docker-compose](https://github.com/purpleteam-labs/purpleteam-s2-containers/blob/main/selenium-standalone/docker-compose.yml)

# Useful commands

## [docker-compose](https://docs.docker.com/compose/reference/)

`docker-compose down`  
Analogous to the `--rm` of `docker run --rm`  
Also check the [flags](https://docs.docker.com/compose/reference/down/)  
By default removes Containers for services defined in the Compose file, networks defined in the networks section of the Compose file, the default network if one is used.

`docker-compose down --rmi all`  
Removes `all` images used by any service

## docker CLI

[Attach](https://docs.docker.com/engine/reference/commandline/attach/) to a running container:  
`docker attach [container]`

[List](https://docs.docker.com/engine/reference/commandline/container_ls/) all running containers:  
`docker container ls`

List all containers:  
`docker container ls --all, -a`

Stop all containers:  
`docker stop $(docker ps -a -q)`

When a docker container won't stop:  
`ps aux | grep docker`  
`sudo killall -9 dockerd`  
`sudo kill -9 <pid of anything with docker in the output>`  
`sudo /etc/init.d/docker start`  

Inspect a container:  
`docker container inspect [container-id]`

Inspect an image:  
`docker image inspect [image_id]`

Remove all containers:  
`docker rm $(docker ps -a -q)`  
or  
`docker container prune` [options](https://docs.docker.com/engine/reference/commandline/container_prune/)

List all images:  
`docker images -a`  
or  
`docker image ls -a`

Remove a specific image:  
`docker rmi [image-name]`

Remove all images:  
`docker rmi $(docker images -a -q)`  
or  
`docker image prune` [options](https://docs.docker.com/engine/reference/commandline/image_prune/)  

Docker history:  
`docker history --no-trunc [image_id]`

Use [dive](https://github.com/wagoodman/dive) to locate files in an image:  
1. Use dive
2. `docker create [image]` returns container ID
3. `docker cp [container_id]:[source_path] [destination_path]` can now view file contents
4. `docker rm [container_id]`

Shell into running container:  
`docker exec -it [container_id] [bash || /bin/ash]`

Tail stdout (and stderr, as [Docker merges stdout and stderr](https://www.scalyr.com/blog/how-to-redirect-docker-logs-to-a-single-file)) of a container, useful for watching Zap, Selenium or any other Slave container logs:  
`docker logs --follow [container-name]`  
If you would like to send those logs to a file as well:  
`docker logs --follow [container-name] |tee output.log$(date '+%Y-%m-%d_%T')`  
If you would like to send those logs to a file without viewing them via your terminal:  
`docker logs --follow [container-name] > output.log$(date '+%Y-%m-%d_%T')`

Shell into a running container once it's running from a docker-compose command:  
`docker-compose exec [service-name-in-docker-compose-file] /bin/ash`  
Or as an npm script:  
~/Source/purpleteam-orchestrator `npm run dc-up-shell-app`  
On the `app-scanner` service of the orchestrator compose file, this was useful for:  
1. Verifying connectivity with `sam local start-lambda` running on the host:  
  From within the container, find the hosts IP address becuase `host.docker.internal` isn't implemented:  
  `/sbin/ip route | awk '/default/ { print $3 }' | awk '!seen[$0]++'`  
  prints `172.25.0.1`  
   * Either install and use curl  
     `su root`  
     `apk --update --no-cache add curl`  
     `curl 172.25.0.1:3001`
   * Or use wget with no install  
     `wget -O - 172.25.0.1:3001`  
2. Verifying connectivity with zap containers (Once the regex in the [docker-compose](https://github.com/purpleteam-labs/purpleteam-s2-containers/blob/main/app-slave/docker-compose.yml) file of purpleteam-s2-containers app-slave was setup correctly, and of course watching what's happening in the zap container with the `docker logs` command), like so:  
  `curl appslave-zap_1:8080/UI`  
  `curl appslave-zap_2:8080/UI`  
  `curl 172.25.0.8:8080/UI`



## Networking

List networks  
`docker network ls`

Inspect specific network  
`docker network inspect <network-name>`

Ip address info  
`ip addr show`

Remove specific network  
`docker network rm <network-name>`

iptables list  
`iptables -L`


## Workings for dockerising Orchestrator

* List all open ports:  
  `ss -lntu`
* `docker build --tag purpleteam-orchestrator-img .`  
  `docker image ls`  
  `docker run -e "NODE_ENV=local" -p 2000:2000 -it --rm --name purpleteam-orchestrator-cont purpleteam-orchestrator-img`
* Find ip address bound to a container  
  `docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' pt-orchestrator-cont`
* Find ip address of host from containers perspective, in alpine
  * Provides shell  
   `docker run -e "NODE_ENV=local" -p 2000:2000 -it --rm --name purpleteam-orchestrator-cont purpleteam-orchestrator-img /bin/sh`
  * Provides IP address  
   `route | awk '/^default/ { print $2 }'`     
* Testing that the orchestrator container can reach the app-tester  
  `apk add --no-cache curl`  
  `curl 172.0.0.1:3000`  
  Or, just use wget out of the box:  
  `wget -O - 172.0.0.1:3000`


## Generic

* Which comtainers are running as root:  
  `docker ps --quiet | xargs docker inspect --format '{{ .Id }}: User={{ .Config.User }}'`  
  Just redis

# Useful resources

[Compose file version 3 reference](https://docs.docker.com/compose/compose-file/)

[Docker cheat sheet](https://github.com/eon01/DockerCheatSheet)

`ARG`, `ENV` and `.env` - [complete guide](https://vsupalov.com/docker-arg-env-variable-guide/)  
`docker-compose config` great for testing what your `docker-compose.yml` file content looks like after the substitution step has been performed

