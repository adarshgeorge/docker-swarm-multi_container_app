## Creating Multi Container App using Flask Container and Redis in Docker Swarm 2 Node Cluster!

* A DevOps Project to display Location of IP address.


![alt HelloWorld](https://github.com/adarshgeorge/docker-multi_container_app/blob/master/png/Search.png)



### Pre-Requisite
An 2 Server/EC2 with docker installed. 

Userdata while launching an EC2 instance.
```
#!/bin/bash
sudo yum install docker -y
sudo service docker restart
sudo chkconfig docker on
sudo usermod -a -G docker ec2-user

```



First we need to initialize the Swarm Mode

```
$ docker swarm init

Swarm initialized: current node (jdza9qu4zxmgswvtqrlzdipxh) is now a manager.
To add a worker to this swarm, run the following command:
   
    docker swarm join --token SWMTKN-1-1pxslguh0yy2i8f9la85po3f53etjyejs7m27jsp7ustkkw6wa-b7s13chwrmyhn1l533zcxc9vm 172.31.6.208:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
$
```

Creating redis cache server!

```
docker run \
-d \
--name ipstack-cache \
--network ipstack-net \
--restart always \
redis:latest
```

Creating Overlay network 
```
$ docker network create --driver overlay ipstack-net
gw6ehdudygxydxvlihbmy8shp
$

```

Now create redis service

```
docker service create \
--name ipstack-cache \
--replicas 1 \
--network ipstack-net \
redis:latest

```


Creating Flask Service


```
docker service create \
--name ipstack-app \
--network ipstack-net \
--replicas 2 \
-p 80:8080 \
-e REDIS_HOST=ipstack-cache \
-e REDIS_CACHE=600 \
-e FLASK_PORT=8080 \
-e IPSTACK_KEY='91e59ddd65394199ad3f905d35532479' \
adarshgeorge999/ipstackapp:1
```

Verify the created services

```
$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                          PORTS
t7drwgzwp6xg        ipstack-app         replicated          2/2                 adarshgeorge999/ipstackapp:1   *:80->8080/tcp
wsvhvt4w2dff        ipstack-cache       replicated          1/1                 redis:latest
$

```
**Browse the Hostname or IP address.**

![alt Out1]() 

Refresh the page to verify app is loading from the different container. 

![alt Search]() 

**Output**

Search any IP

![alt Search]() 


Now again search the same IP address to verify the IP address is cached in reddis container. 

![alt Out2]() 


## That's it!!