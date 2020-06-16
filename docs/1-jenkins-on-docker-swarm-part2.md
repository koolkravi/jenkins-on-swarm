

docker build -t 127.0.0.1:5000/vdp/jenkins:2.228 .
docker push 127.0.0.1:5000/vdp/jenkins:2.228

```
sudo docker node update --label-add JENKINS=true --label-add bar=baz   ubuntuvm02       // 10.0.0.7
sudo docker  inspect node ubuntuvm02
```

```
docker stack deploy -c jenkins.yml --with-registry-auth jenkins
```

```
docker exec db6c65726115 cat /var/jenkins_home/secrets/initialAdminPassword    //00838d27dd674c45ae1e40e9d94ccf37
```

```
docker exec db6c65726115 bash -c 'echo "$HTTP_PROXY"'
```

```
curl https:/127.0.0.1:9008/jenkins
```

```
sudo docker stack deploy -c database.yml --with-registry-auth jenkins
```




# ref
- http://hub.docker.com.
- https://medium.com/better-programming/about-var-run-docker-sock-3bfd276e12fd

docker stack ls
docker stack services jenkins
docker stack ps jenkins 

docker stack rm jenkins 
docker volume rm  jenkins_data

docker service logs -f  jenkins_app-server

docker logs <container id>

check proxy 
docker container run --rm busybox env

docker exec <container_id> bash -c 'echo "$HTTP_PROXY"'
