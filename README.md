# IPGeolocation-using-docker-swarm
Creating IPGeolocation using docker swarm

## Description

IP-based geolocation is a way that you can find the location of an internet-connected computing or mobile device. To get started, all you need is your target’s IP address and a geolocation lookup tool. A geolocation lookup tool canvasses public databases to determine the contact and registration information for a particular IP address. With both of these tools in hand, you simply input the IP address into the geolocation lookup tool and you will receive the location of your target.

It is same as my previous git article regarding [IPgeolocation-cache-using-Multi-Container](https://github.com/devasivaram/IPgeolocation-cache-using-Multi-Container.git), But here we use docker swarm and in docker swarm we use service instead of container. Also, here we use 1 master with 2 worker instances.

## Pre-requests
- A running 3 Amazon Linux instance, in which 1 with docker installed
- A domain name which points to the EC2 instance
- Get API key from: [API_KEY](https://app.ipgeolocation.io/auth/login)

##Procedure

## Step 1 - Initialising docker swarm

We need to initialize docker swarm in master server so that we can use that generated docker swarm join token to connect 2 worker instances.

~~~sh
# docker swarm init
~~~

>Result:
>![image](https://user-images.githubusercontent.com/100773863/164956899-24d4d4ae-6ede-4315-8b4d-4be80f48bfcb.png)

***Now, run this generated join token in both of your worker instnaces:***

>Result:
>![image](https://user-images.githubusercontent.com/100773863/164956922-b804b637-0912-4aa0-83eb-97f0a5b63afc.png)

***To verify the workers has been added to master, run this command:***

~~~sh
# dokcer node ls
~~~

>Result:
>![image](https://user-images.githubusercontent.com/100773863/164956995-dfeab358-a755-45bd-a480-295a27ec98c3.png)

## Step 2 - Drain master instance

While we create a service in master, the master itself acvt as an worker if we don't drain master server. So to drain master server, we need to change the availablity state to drain:

~~~sh
# docker node update --availability drain {master-server-hostname}
# docker node ls
~~~

>Result:
>![image](https://user-images.githubusercontent.com/100773863/164957091-3747bd24-e7e4-4a42-b49e-66eaf507835f.png)

## Step 3 - Label the worker instances

We use Labels to add extra metadata to your docker images, service, networks, swarm nodes etc. Once added these label(s) allows you to filter your docker resources.

> Here I use 2 worker instnaces so am adding lables for those two. You can use 3 worker instances, like 1 for redis image, 1 for api-service image, and 1 for frontend image.

~~~sh
# docker node update --label-add resource=cache {worker-server-hostname}
# docker node update --label-add resource=disk {worker-server-hostname}
~~~

>Result:

>![image](https://user-images.githubusercontent.com/100773863/164957554-c8185e23-fad2-46f6-8e8a-66555e9bb776.png)
>![image](https://user-images.githubusercontent.com/100773863/164957542-aede71f1-e0cc-4711-86bd-8a897109c398.png)


## Step 4 - Create an network 

Crewate an networtk with overlay so that our worker instances can communicate each other

~~~sh
# docker network create --driver overlay iplocation-net
~~~

## Step 5 - Create services

### To create redis

~~~sh
# docker service create \
--constraint 'node.labels.resource == cache' \
--network iplocation-net \
--name iplocation-cache-service \
--replicas 1 \
redis:latest
~~~

Result:
>![image](https://user-images.githubusercontent.com/100773863/164957656-a2f5cf21-8c13-4ade-81b1-3a5fbcb0b992.png)


### To create api-service:

~~~sh
# docker service create \
--name iplocation-api-service \
--network iplocation-net \
--replicas 4 \
--constraint 'node.labels.resource == disk' \
-e REDIS_PORT="6379" \
-e REDIS_HOST="iplocation-cache-service" \
-e APP_PORT="8080" \
-e API_KEY_FROM_SECRETSMANAGER=False \
-e API_KEY="your-API-KEY" \
devanandts/apiimage:custom
~~~

Result:
>![image](https://user-images.githubusercontent.com/100773863/164957733-b8354257-8496-48a1-ba2a-e59fe3c9f2b8.png)


### To create frontend service:

~~~sh
# docker service create \
--name iplocation-frontend-service \
--network iplocation-net \
--replicas 4 \
--constraint 'node.labels.resource == disk' \
-e API_SERVER="iplocation-api-service" \
-e API_SERVER_PORT="8080" \
-e API_PATH="/api/v1/" \
-e APP_PORT="8080" \
-p 80:8080 \
devanandts/frontimagenew:custom
~~~

Result:
>![image](https://user-images.githubusercontent.com/100773863/164957804-b787955a-4014-4f59-9d30-f0cbcd8ced47.png)


Now our setup is complete and we can check the IP using: api.devanandts.tk/ip/required-ip. 
Example: api.devanandts.tk/ip/8.8.8.8

Result:
>![image](https://user-images.githubusercontent.com/100773863/164957866-9c78dc3f-e816-4a3c-bcd2-e8a9ad77b720.png)

## Conclusion
This is how we create ipgeolocation-cache service using docker-swarm. Please contact me when you encounter any difficulty error while using this terrform code. Thank you and have a great day!


### ⚙️ Connect with Me
<p align="center">
<a href="https://www.instagram.com/dev_anand__/"><img src="https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white"/></a>
<a href="https://www.linkedin.com/in/dev-anand-477898201/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a>


