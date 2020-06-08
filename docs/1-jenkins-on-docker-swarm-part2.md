
1. Jenkins 

docker volume create -d local-persist -o mountpoint=/opt/jenkins/data/jenkins_home/ --name=jenkins-home

docker build -t myjenkins .
docker stop jenkins-master
docker rm jenkins-master
docker run -p 8080:8080 -p 50000:50000 --name=jenkins-master -v jenkins-home:/var/jenkins_home -d myjenkins

2. nginx







# ref
- http://hub.docker.com.