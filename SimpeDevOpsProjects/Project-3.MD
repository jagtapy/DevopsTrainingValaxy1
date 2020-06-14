## Simple DevOps Project -3 


1. Launch an EC2 instance for Docker host

2. Install docker on EC2 instance and start services 
  ```sh 
  yum install docker
  service docker start
  ```

3. create a new user for Docker management and add him to Docker (default) group
```sh
useradd dockeradmin
passwd dockeradmin
usermod -aG docker dockeradmin #add the dockeradmin user to docker group

```

4. Write a Docker file under /opt/docker

```sh
mkdir /opt/docker

### vi Dockerfile
# Pull base image 
From tomcat:8-jre8 

# Maintainer
MAINTAINER "valaxytech" 

# copy war file on to container 
COPY ./webapp.war /usr/local/tomcat/webapps
```

5. Login to Jenkins console and add Docker server to execute commands from Jenkins  
install plugin publish over ssh
Manage Jenkins --> Configure system -->  Publish over SSH --> add Docker server and credentials

On your docker server
```sh
vi /etc/ssh/sshd_config
update as below
PasswordAuthentication yes
#PermitEmptyPasswords no
#PasswordAuthentication no

then service sshd restart

```
6. Create Jenkins job 

A) Source Code Management  
 Repository : https://github.com/ValaxyTech/hello-world.git  
 Branches to build : */master  

B) Build
 Root POM: pom.xml  
 Goals and options : clean install package  
 
C) send files or execute commands over SSH
 Name: docker_host  
 Source files	: `webapp/target/*.war`
 Remove prefix	: `webapp/target`
 Remote directory	: `//opt//docker`  
 Exec command[s]	: 
  ```sh
  docker stop valaxy_demo;
  docker rm -f valaxy_demo;
  docker image rm -f valaxy_demo;
  cd /opt/docker;
  docker build -t valaxy_demo .
  ```

D) send files or execute commands over SSH  
  Name: `docker_host`  
  Exec command	: `docker run -d --name valaxy_demo -p 8090:8080 valaxy_demo` 
 # -d is for detached mode
 #be default it runs on 8080 ; so we are exposing 8090
 #--name valaxy_demo here we are mentioning name of container and at the end it is name of image
  

7. Login to Docker host and check images and containers. (no images and containers)
before running the job change the permission as below 
 ```sh
chown -R dockeradmin:docker /opt/docker
 ```
8. Execute Jenkins job
#i got below error after execution
#Maven installation needs to be available for this project to be built.Either your server has no Maven installations defined, or the #requested Maven version does not exist.

#solution was to go to Manage Jenkins -> Global Tool Configuration, and set up maven there.
#just added Name: my_maven_3.3.9

9. check images and containers again on Docker host. This time an image and container get creates through Jenkins job

10. Access web application from browser which is running on container
```
<docker_host_Public_IP>:8090
```
