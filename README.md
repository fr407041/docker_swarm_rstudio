Docker Swarm Cluster Rstudio
==


[![hackmd-github-sync-badge](https://hackmd.io/vOv3_xPrSNihH39_kR8ZKQ/badge)](https://hackmd.io/vOv3_xPrSNihH39_kR8ZKQ)
### Index

1. [Build docker swarm cluster](#docker_swarm)
1. [Build docker stack cluster service](#docker_stack)


----
<a name="docker_swarm"/>

##### 1.Build docker swarm cluster

1. In IP_1 launch docker a docker swarm manager
```
docker swarm init --advertise-addr IP_1
```
2. At IP_1 Check Swarm worker's token 
```
docker swarm join-token -q worker
```
3. SSH IP_2 & IP_3 & set it as `worker`
```
docker swarm join --token xxxx IP_1:2377
```
4. At IP_1 check docker swarm cluster status (`It shoulde be like below : one manager and one worker`)
```
docker node ls
```
![](https://i.imgur.com/i4iFpvi.png)

##### 2. Build docker stack cluster service

* Create docker-stack.yml (`yaml is as below`)
> Note: `mode:` must be `global`, and IP_1 & IP_2 will create only one rstudio container service (`mode:` can't be `replicated`)

```
version: '3.6'
networks:
  rnet:
    driver: "overlay"  
services:
  rstudio01:
    image: rocker/rstudio:latest
    hostname: "docker_{{.Node.Hostname}}-{{.Service.Name}}"
    environment:
      - PASSWORD=plz write your password
    ports:
      - target: 8787
        published: 8788
        protocol: tcp
        mode: host  
    deploy:
      mode: global
      endpoint_mode: dnsrr 
      placement:
        constraints:
          - node.role == worker   
      restart_policy:
        condition: on-failure
```

1. Check current service list
```
docker service ls
```
![](https://i.imgur.com/p3HK24f.png)

2. Check rstudio service
```
docker service ps ID 
```
![](https://i.imgur.com/VvzA92h.png)

3. Link and Check Rstudio Service (`Result as below`)
* http://IP_1:8788 have no rstudio service
* http://IP_2:8788 have rstudio service
* http://IP_3:8788 have rstudio service

4. You have successful deploy each worker with only one rstudio service by Parameter `mode: global`

* Note: mode cannot be `replicated`, it will result conflict if two replicates are in the same worker node(As below). In addition, deploy must add parameter `endpoint_mode: dnsrr` and port setting must write as.
```
- target: 8787
  published: 8788
  protocol: tcp
  mode: host  
```

![](https://i.imgur.com/Dl9h550.png)


![](https://i.imgur.com/NIg6Ygd.png)


###### tags: `rstudio` `docker swarm`