Setup jenkins  on Docker Swarm

# Pre-requisite

## Create 3 vm 
```
vagrant up

	ips
	10.0.0.5
	10.0.0.6
	10.0.0.7
```
	
## Setup NFS Server and client (Ref: 1-nfsmount.md)
 
## Pre-requisite [https://github.com/koolkravi/docker-swarm/blob/master/docker-swarm-deploy-stack-part2.md]
 
## Install docker  (2-install-docker.sh)

## Install Docker Compose  (3-install-docker-compose.sh)

## Run docker without sudo
```
sudo setfacl -m user:$USER:rw /var/run/docker.sock
```

## Setup swarm Manager and workder node (4-create_swarm_manager_and_add_worker.md)

## Setup a docker registry (5-docker-registry.md) from master node 10.0.0.6

# Jenkins Installation Steps

## Step 1:  Create Docker file
```
jenkins/Dockerfile
```

## Step 2:  Build and Run
```
docker build -t myjenkins .
docker stop jenkins-master
docker rm jenkins-master
docker run -p 8080:8080 --name=jenkins-master -d myjenkins
```

### tail log file
```
docker exec jenkins-master tail -f /var/log/jenkins/jenkins.log
```

### Retrieve logs if jenkins crashes
```
docker stop jenkins-master
docker cp jenkins-master:/var/log/jenkins/jenkins.log jenkins.log
cat jenkins.log
```

## Step 3: Persist Docker data with volume

### Approach 1
```
#docker volume create jenkins-data
```

```
docker stop jenkins-master
docker rm jenkins-master
docker run -p 8080:8080 -p 50000:50000 --name=jenkins-master --mount source=jenkins-log,target=/var/log/jenkins --mount source=jenkins-data,target=/var/jenkins_home -d myjenkins
docker exec jenkins-master tail -f /var/log/jenkins/jenkins.log
```

```
docker exec jenkins-master cat /var/log/jenkins/jenkins.log
docker exec jenkins-master ls /var/cache/jenkins/war
```

### copy data if you have lost the container
```
docker run -d --name mylogcopy --mount source=jenkins-log,target=/var/log/jenkins debian:stretch
docker cp mylogcopy:/var/log/jenkins/jenkins.log jenkins.log
```

### Approcah 2: NFS Mount point as persistent volume   
```
docker volume create -d local-persist -o mountpoint=/opt/jenkins/data/jenkins_home/ --name=jenkins-home
```

```
docker stop jenkins-master
docker rm jenkins-master

docker run -p 8080:8080 -p 50000:50000 --name=jenkins-master -v jenkins-home:/var/jenkins_home -d myjenkins

#docker run -p 8080:8080 -p 50000:50000 --name=jenkins-master --mount source=jenkins-log,target=/var/log/jenkins -v jenkins-home:/var/jenkins_home -d myjenkins
````

## Step 4: NGINX Proxy in front of jenkins
```
/jenkins-nginx/Dockerfile
```

```
/conf/nginx.conf
/conf/jenkins.conf
```

```
docker build -t myjenkinsnginx jenkins-nginx/.
```

### MAKE A DOCKER NETWORK SO NGINX CAN TALK TO JENKINS

We want to create a network between our two containers so that they can easily find each other. One reason they’ll be able to easily find each other is Docker networks offer something they call “automatic service discovery.” Which is fancy speak for creating DNS names on the network that match the container names you create. 
This is why our NGINX config file references jenkins-master. Docker will handle making that DNS entry for us when our container attaches to the network

```
docker network create --driver bridge jenkins-net
docker network ls
```

### BUILD THE NGINX IMAGE AND LINK IT TO THE JENKINS IMAGE
```
docker stop jenkins-master
docker rm jenkins-master
```

### restart our Jenkins master container, but attach it to the network this time
```
docker run -p 8080:8080 -p 50000:50000 --name=jenkins-master --network jenkins-net --mount source=jenkins-log,target=/var/log/jenkins --mount source=jenkins-data,target=/var/jenkins_home -d myjenkins
```

### build start the NGINX container and attach it to jenkins-master
```
docker build -t myjenkinsnginx jenkins-nginx/.
docker run -p 80:80 --name=jenkins-nginx --network jenkins-net -d myjenkinsnginx
```

### TEST
```
http://localhost
curl http://localhost:8000 
```

### JENKINS IMAGE CLEANUP

e have NGINX listening on port 80, we don’t need the Jenkins image or container to expose port 8080. 
Let’s remove that exposure by removing the port option when we start the container

```
docker stop jenkins-master
docker rm jenkins-master
docker run -p 50000:50000 --name=jenkins-master --network jenkins-net --mount source=jenkins-log,target=/var/log/jenkins --mount source=jenkins-data,target=/var/jenkins_home -d myjenkins
```

### Test
```
curl http://localhost
```

## Step 4: DOCKER COMPOSE AND JENKINS

###  Compose file
```
docker-compose.yml
```

### Edit jenkins.conf file inside /jenkins-nginx/jenkins.conf 

Compose naming standard: [project]_[service]_[instance].

```
proxy_pass         http://jenkins-master:8080; 

to

proxy_pass         http://jenkins_master_1:8080;
```

### Build and run 

Clean
```
docker stop jenkins-nginx
docker rm jenkins-nginx
docker stop jenkins-master
docker rm jenkins-master
docker volume rm jenkins-data
docker volume rm jenkins-log
docker network rm jenkins-net
```

```
docker-compose build
docker-compose -p jenkins up -d
```

-p option, give “project” a name, jenkins

```
docker-compose -p jenkins ps
```

### Docker Compose tears down your network (because it can be easily recreated) but doesn’t remove your volume. When it comes up again it recreates the network and doesn’t bother with the volume
```
docker-compose -p jenkins down
```

to delete volume as well the pass -v option
```
docker-compose -p jenkins down -v
```

### TEST

```
curl http://localhost
```






# Ref
## https://technology.riotgames.com/news/putting-jenkins-docker-container

```
docker pull jenkins/jenkins
docker run -p 8080:8080 --name=jenkins-master jenkins/jenkins
curl http://localhost:8080

### run ad DAEMON
docker rm jenkins-master
docker run -p 8080:8080 --name=jenkins-master -d jenkins/jenkins

### MEMORY SETTINGS
docker stop jenkins-master
docker rm jenkins-master
docker run -p 8080:8080 --name=jenkins-master -d --env JAVA_OPTS="-Xmx8192m" jenkins/jenkins

### INCREASING THE CONNECTION POOL
docker stop jenkins-master
docker rm jenkins-master
docker run -p 8080:8080 --name=jenkins-master -d --env JAVA_OPTS="-Xmx8192m" --env JENKINS_OPTS=" --handlerCountMax=300" jenkins/jenkins

### running basic commnad in container
docker exec jenkins-master ps -ef | grep java
```

## Defaulkt docker file : https://github.com/Jenkinsci/docker/blob/master/Dockerfile
## https://github.com/jenkinsci/docker/blob/master/README.md
## Jenkins behind an NGinX reverse proxy: https://wiki.jenkins.io/display/JENKINS/Jenkins+behind+an+NGinX+reverse+proxy
## Docker network : https://docs.docker.com/network/
## How to Do Basic Debugging With Docker Compose : https://www.matthewsetter.com/basic-docker-compose-debugging/
```
docker-compose -p jenkins up -d
docker-compose -p jenkins ps
docker-compose -p jenkins logs --follow nginx
docker compose -p jenkins up -d build nginx
docker-compose -p je nkins down 
```
