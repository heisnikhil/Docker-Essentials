# Docker-Essentials
Docker Essentials 

----------------------------------------------------------------------
Lab 1: Creating an EC2 Instance in AWS and Installing Docker
ubuntu-latest
free tier
t2.micro
----------------------------------------------------------------------

##Installing Docker on Ubuntu operating system 

sudo hostnamectl set-hostname docker
bash
sudo su
apt update -y
apt install curl -y
curl -SSL https://get.docker.com/ | sh
service docker status
usermod -aG docker ubuntu   # regular user i.e ubuntu
docker --version

-----------------------------------------------------------------------
Lab 2: Basic Docker Commands
-----------------------------------------------------------------------
Task 1: Creating your first Docker container

docker run hello-world

Task 2: Basic Commands to run the Container in Interactive mode

docker pull ubuntu
docker image ls
docker run -it --name ct1 ubuntu
touch f1 f2 f3
ls
exit
docker ps
docker ps -a
docker run -it --name ct2 ubuntu

Press Crtl+P+Q to switch the terminal to Docke Host.

docker ps
docker exec -it ct2 /bin/sh
exit
docker ps
docker attach ct2
exit
docker ps
docker ps -a

Task 3: Port Mapping from Docker Host to container

docker run -d -p 80:80 httpd
docker ps
docker exec -it < replace container id/name > /bin/bash
exit
docker ps
docker exec -it < replace container id/name > /bin/bash
kill 1
docker ps -a
docker container stop < replace container id/name >
docker container rm < replace container id/name >
docker image ls
docker image rm < replace image id >

------------------------------------------------------------------------------------
LAB-3
------------------------------------------------------------------------------------
Task 1: Docker Lifecycle 

docker pull httpd
docker image ls
docker image history httpd
docker container create httpd
docker container ls -a
docker container start < replace container id/name >
docker container ls
docker container stop < replace container id/Name >
docker container ls -a
docker container start < replace container id/Name >
docker container pause < replace container id/Name >
docker container ls -a
docker container unpause < replace container id/Name >
docker container ls -a
docker exec -it < replace container id/name > bash
cd htdocs
apt update
apt install wget -y
rm index.html
echo "Welcome to Docker Training" > index.html
exit
docker commit < replace container id/name > myhttpd:version1
docker image ls
docker run -d -p 8080:80 myhttpd:version1
curl < public IP>:8080
docker container ls
docker logs < replace container id/name >
docker stats < replace container id/name >
docker container ls
docker stop < replace container id/name >
docker container rm < replace container id/name >
docker image ls
docker image rm < replace image id/name > < replace image id/name >
docker image ls
docker image ls -a




------------------------------------------------------------------------------------
Lab 4: Working with volume mounts in Docker
------------------------------------------------------------------------------------
Task 1: Starting Docker Containers Bind Mounts

mkdir /home/ubuntu/share
echo 'Hello From Docker Host' > /home/ubuntu/share/index.html
docker run -it --name container1 -p 80:80 -v /home/ubuntu/share:/var/www/html ubuntu:18.04 /bin/bash
apt-get update -y && apt-get install apache2 -y
service apache2 start
service apache2 status
echo 'Hello From Container1' > /var/www/html/index.html 
Press Ctrl+P+Q, to switch back to Host
docker run -it --name container2 -v /home/ubuntu/share:/var/www/html ubuntu:18.04
echo 'Hello From Container2' > /var/www/html/index.html 
exit
docker ps -a
docker rm -vf container1 container2 

Task 2: Create a bind mount with --mount option and verify it

docker run -d -it --name newbind01 --mount type=bind,source=/home/ubuntu/share/,target=/app nginx:latest
docker inspect newbind01
mount | grep -i /app


------------------------------------------------------------------------------------
Lab 5: Volume Mounting with Docker Containers
------------------------------------------------------------------------------------ 

Task 1: Creating a new docker volume and inspecting containers


docker volume create ct-volume1
docker volume ls
docker volume inspect ct-volume1

Task 2: Launching a Nginx container mapped to a specific docker volume and verification

docker run -d -p 80:80 --name=nginx-container --mount src=ct-volume1,dst=/usr/share/nginx/html nginx
docker ps
docker container inspect nginx-container
ls /var/lib/docker/volumes/ct-volume1/_data/ 
cd /var/lib/docker/volumes/ct-volume1/_data/ 
touch f1 f2 f3
vi /var/lib/docker/volumes/ct-volume1/_data/index.html

Task 3: Deleting container and attaching the volume to another container

docker container stop nginx-container
docker rm container nginx-container
docker ps -a

docker run -it --name busybox-container1 --mount source=ct-volume1,target=/data busybox sh
ls
cd /data
ls
exit

docker stop busybox-container1
docker rm busybox-container1
docker ps -a

docker volume ls
docker volume rm ct-volume1
docker volume ls

Task 4: Create a container with tmpfs mount and verify it

docker run -d -it --name tmpmount --mount type=tmpfs,destination=/app nginx:latest
docker container inspect tmpmount



------------------------------------------------------------------------------------
Lab 6: Docker Networking 
------------------------------------------------------------------------------------

Task 1: Create a new docker bridge and check connectivity between containers of same bridge

docker network ls
docker network create --driver bridge ct-bridge1
docker network inspect ct-bridge1
docker network ls
docker run -it --network ct-bridge1 --name=ct-c1 busybox
Press Ctrl+P+Q, to switch back to Host
docker run -it --network ct-bridge1 --name=ct-c2 busybox
Press Ctrl+P+Q, to switch back to Host
docker network inspect ct-bridge1
docker ps
docker attach ct-c2
ip addr
ping -c 5 ct-c1
Press Ctrl+P+Q, to switch back to Host


Task 2: Create a new docker bridge and check connectivity between containers of different bridges

docker network create --driver bridge ct-bridge2
docker run -it --network ct-bridge2 --name=ct-c3 busybox
Press Ctrl+P+Q, to switch back to Host
docker run -it --network ct-bridge2 --name=ct-c4 busybox 
Press Ctrl+P+Q, to switch back to Host
docker attach ct-c4
ping -c 5 ct-c3
ip addr
ping -c 5 ct-c1
ping -c 5 ct-c2
Press Ctrl+P+Q, to switch back to Host


Task 3: Using 'Docker network connect' command create a successful connection between containers of different bridges

docker network ls
docker network connect ct-bridge2 ct-c1
docker network inspect ct-bridge2
docker attach ct-c1
ping -c 5 ct-c4
ip addr
ip route

Task 4: Launch a container to host network

docker run -it --network host --name=ct-c5 busybox
ip addr
ifconfig
Press Ctrl+P+Q, to switch back to Host
docker network inspect host

Task 5: Launch a container to none network 

docker run -it --network none --name=ct-c6 busybox
ip addr
Press Ctrl+P+Q, to switch back to Host

------------------------------------------------------------------------------------
Lab 7 : Building a Dockerfile to setup an Ubuntu container with WordPress application
------------------------------------------------------------------------------------
Task 1: Deploying MySQL and WordPress containers

mkdir wordpress
cd wordpress
vi Dockerfile

#Content of Dockerfile to paste

FROM ubuntu:18.04
MAINTAINER ADMIN "admin@cloudthat.com"
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && \
apt-get -q -y install apache2 \
php7.2 \
php7.2-fpm \
php7.2-mysql \
libapache2-mod-php7.2
ADD http://wordpress.org/latest.tar.gz /tmp
RUN tar xzvf /tmp/latest.tar.gz -C /tmp  \
&& cp -R /tmp/wordpress/* /var/www/html
RUN rm /var/www/html/index.html && \
chown -R www-data:www-data /var/www/html
EXPOSE 80
CMD ["/bin/bash","-c","service apache2 start && sleep 5000"]


#End of Dockerfile

# You can download the above Dockerfile from S3 using - wget https://hpe-content.s3.ap-south-1.amazonaws.com/Dockerfile

docker build -t ct-wordpress:v1 .
docker image ls
docker network create --driver bridge ct-bridge 
docker run -d --network ct-bridge --name mysql -e MYSQL_DATABASE=wordpress -e MYSQL_USER=admin -e MYSQL_PASSWORD=password -e MYSQL_ROOT_PASSWORD=password mysql:5.7

docker ps 
docker run -d --network ct-bridge -p 80:80 ct-wordpress:v1
docker ps

------------------------------------------------------------------------------------
Lab 8: Pushing the image to DockerHub
------------------------------------------------------------------------------------
Pushing the image to DockerHub

docker login
docker tag ct-wordpress:v1 <replace your dockerhub account name>/ct-wordpress:v1
docker image ls

docker push <replace your dockerhub account name>/ct-wordpress:v1

docker image ls
docker image rm <replace your dockerhub account name>/ct-wordpress:v1 ct-wordpress:v1
docker run -d -p 8080:80 <replace your dockerhub account name>/ct-wordpress:v1
docker stop < replace container id/name >
docker container rm < replace container id/name >
docker image ls
docker image rm < replace image id/name > < replace image id/name >
docker image ls
docker image ls -a

====================================================================================
