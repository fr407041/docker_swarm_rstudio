### Index

1. [Build docker swarm cluster](#docker_swarm)
1. [Build docker stack cluster service](#docker_stack)


----
<a name="docker_swarm"/>

##### 1.Build docker swarm cluster

1. In IP1 launch docker a docker swarm manager
```
docker swarm init --advertise-addr IP1
```
2. At IP1 Check Swarm worker's token 
```
docker swarm join-token -q worker
```
3. SSH IP2 & set it as work
```
docker swarm join --token xxxx IP1:2377
```
4. At IP1 check docker swarm cluster status
```
docker node ls
```

##### 2. Build docker stack cluster service

* Create docker-stack.yml (`Content as below`)

```
version: '3.6'
networks:
  rnet:
    driver: "overlay"  
services:
  rstudio01:
    image: my_rstudio:0.0.1
    hostname: rserver
    environment:
      - USER=plz write your account
      - PASSWORD=plz write your password
    ports:
      - 8787:8787
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == worker   
      restart_policy:
        condition: on-failure
```
