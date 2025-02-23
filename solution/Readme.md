I am trying this demo on my Amazon EC2 instance.

1. For docker and docker-compose installation, please use the below commands :
yum install docker -y

To install docker-compose :
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

To test if its installed or not use docker-compose version

2. Ran the below commands as mentioned in tutorials :

systemctl start docker  - To start docker

docker pull infracloudio/csvserver:latest

Status: Downloaded newer image for infracloudio/csvserver:latest
docker.io/infracloudio/csvserver:latest

docker pull prom/prometheus:v2.22.0
Status: Downloaded newer image for prom/prometheus:v2.22.0
docker.io/prom/prometheus:v2.22.0

3. First of all install git using below command :
yum install git -y

To clone the git repository :
git clone https://github.com/infracloudio/csvserver.git

To create a new private repository
git init

Now create a new folder called solution and cd into it
mkdir solution
cd solution

Part 1 :

1. docker run infracloudio/csvserver:latest
2021/06/13 15:59:13 error while reading the file "/csvserver/inputdata": open /csvserver/inputdata: no such file or directory

Got the above error. To resole it run the container in detatched mode using -d
docker run -d infracloudio/csvserver:latest
output : 0f1ef398005897244aa13a69a9c9dfa7701951ff7fbbf7caa4bc582c8eff778b

To check the status of container use :
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS                      PORTS     NAMES
0f1ef3980058   infracloudio/csvserver:latest   "/csvserver/csvserver"   42 seconds ago   Exited (1) 41 seconds ago 

3. Now for the bashscript gencsv.sh
#!/bin/bash
# Amaresh
touch inputFile
echo $'0, 234\n1, 98\n2, 34' > inputFile

This is used to create a file called inputFile and then print the output

Before running we have to give executable permission and then run the shell script :
[root@ip-172-31-4-187 solution]# chmod a+x gencsv.sh
[root@ip-172-31-4-187 solution]# ./gencsv.sh


4.Now to resolve the error while running the container in detatched mode :
docker run -d -v /root/solution/inputFile:/csvserver/inputdata infracloudio/csvserver:latest

Now as we have mapped the volume, when we try running the docker ps we get below output:
[root@ip-172-31-4-187 solution]# docker ps
CONTAINER ID   IMAGE                           COMMAND                  CREATED         STATUS         PORTS                            NAMES
6729cbf77b0f   infracloudio/csvserver:latest   "/csvserver/csvserver"   4 seconds ago   Up 2 seconds   9300/tcp                         upbeat_chatelet

To get shell access of container use below command :
docker exec -it 6729cbf77b0f /bin/bash

To know which port the application is listening
[root@6729cbf77b0f csvserver]# netstat -tulpn | grep LISTEN
tcp6       0      0 :::9300                 :::*                    LISTEN      1/csvserver

To change the default port for the application to 9393:
docker run -d -v /root/solution/inputFile:/csvserver/inputdata -p 80:9393 infracloudio/csvserver:latest

For stopping the container use the below command :
docker stop 6729cbf77b0f

Part 2:

For removing/deleting container :
docker rm 6729cbf77b0f

Create a docker-compose.yaml file for the setup from part I.
vi docker-compose.yaml

version: "3"
services:
   mycsvserver:
     image: infracloudio/csvserver:latest
     container_name: csvserver_container
     ports:
       - 80:9393
     volumes:
       - /root/solution/inputFile:/csvserver/inputdata

As port 9393 is already allocated we are getting the below output :
Creating network "solution_default" with the default driver
Creating csvserver_container ...
Creating csvserver_container ... error

ERROR: for csvserver_container  Cannot start service mycsvserver: driver failed programming external connectivity on endpoint csvserver_container (4b5985dad2e527370a091e1f0fbb1d3a93ac4736474ed30270e8204add8dcef2): Bind for 0.0.0.0:80 failed: port is already allocated


So we would try allocating a different port 9001and the local port to 8001 - changing the port section in docker-compose.yaml
ports:
       - 8001:9001

[root@ip-172-31-4-187 solution]# docker-compose up
Removing csvserver_container
Recreating 0fbe0137e337_csvserver_container ... done
Attaching to csvserver_container
csvserver_container | 2021/06/14 04:57:47 listening on ****
 So this creates a new container.
