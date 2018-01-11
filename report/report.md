# AIT Lab 04

### Introduction
In this lab, we are going to get familiar with dynamic scaling with Docker. It is based on the load balancing lab.

### Task 0
[M1]: No, the current solution would not be suited for a production environment. The main problems are the manual launching of the containers. It would be a working system but the manual launching would not make it suitable for a production environment.

[M2]: We would have to modify the configuration file so that a third web app container will be launched along with the first two. More precisely the Docker image will be instancied a third time to create the third container.

[M3]: The problem with this current solution is that the configuration and the launching of the containers are all to be manually done. A better solution would be to find a way to automate everything so that it does not need any admin to ensure that the whole system works well. 

[M4]: We could let the load balancer determine the number of web application nodes to launch depending on the incoming traffic.

[M5]: It is not possible with our current solution to run additional management processes as we are following the general principle of Docker, namely "one process per container". We would have to add a supervisor to be able to run several processes inside one container.

[M6]: To add more web server nodes, we would have to add more lines like the ones beginning with sed. It is not dynamic as we would have to do that manually for each web server nodes we want to add. A solution would be to give the number of webapp to create everytime some change happens and make it create them dynamically?

*Deliverables:*

1. Screenshot of the stats page of HAProxy at this point:

    ![](https://i.imgur.com/h86NYuH.png)
    
2. URL of our repository for this lab: https://github.com/Thuy-My/Teaching-HEIGVD-AIT-2016-Labo-Docker

### Task 1

1. Screenshot of the stats page of HAProxy:

    ![](https://i.imgur.com/Z1bysPk.png)
    
2. The principle of the Docker design is to run only one process per container. <!--Additionally, the container automatically stops once the foreground stops.-->
    In our case, we want to break this principle and to have several processes running in the same container. To do so, we need to use an init system whose role is to manage daemons and coordinates the boot process.
    The init system that we are using is called `S6` which is a small suite of programs for UNIX and will allow us to supervise processes. More precisely, we are only using the `s6-overlay` scripts. These scripts will automatically integrates s6 into Docker images, which is what we want.
    
    Now all we need to put everything in place. The first thing to do is to add an instruction in the Dockerfile to make it download and install `S6 overlay` during the building of the corresponding image. Then we have to configure `S6` as the main process instead of the old one by adding another instruction: `ENTRYPOINT ["/init]`. The `ENTRYPOINT` instruction is the way to specify which executable has to be run when a container is started from an image. As we can see, now the executable to be ran first is `init` from `S6` and no longer `/scripts/run.sh`.
    
    At this point, the supervisor is well configured and is running. The only problem is that no application is running anymore as we just replaced it with the supervisor process. To correct that, we have to create starting scripts for `S6` so that our applications can be available.
    
    Now everything works. We successfully modified the Docker images to make it support several processes running in the same container, which is originally something that we could not do.
    

### Task 2

1. See the `logs/task2` folder in the repository.

2. We can manage the web app nodes dynamically as opposed to statically as in the initial condition by using Serf. Thanks to Serf, the load balancer will know when a node goes down or goes up and will be able to adapt itself.
    The web app nodes can therefore be managed in a more dynamic fashion.

3. Serf is a tool aimed for cluster membership, failure detection and orchestration. It uses a gossip protocol, which means that every messages are brodcasted to the entire cluster. By consequence every time a web app node will come up or go down, Serf will alert every other component of the cluster. More importantly the message will reach the load balancer who will in turn adapt itself to the new situation by redirecting incoming requests to the new web app in the case where a new one pops up or by redirecting them to only the remaining ones in the case where one web app dies. 
   Another solution would be to use Consul, as it is a distributed service discovery.

### Task 3

1. See the `logs/task3` folder in the repository.

2. See the `logs/task3` folder in the repository.

### Task 4

1. A Docker image is built up from a series of layers. Each layer represents an instruction in the image’s Dockerfile.
The layers are stacked on top of each other and each layer is kept in memory. As soon as we add a new instruction 
the reconstruction of the image will take some time because there is no layer corresponding to this instruction and 
docker creates it.
To optimize a Dockerfile we need to group the commands such as RUN command 1 && command 2 && command 3.

2. We propose to use this command in the dockerfile RUN apt-get update & apt-get install xz-utils to reuse as 
   much as possible what we have done.

3. See the `logs/task4` folder in the repository.

4. We notice that all three files have the same structure, the only difference being the container ID and the IP address. Everytime a new container is ran, the whole haproxy.cfg is replaced. It would have been better if every output was being concatenated instead, that way we can see clearly which containers are in the cluster.

### Task 5

1. See the `logs/task5` folder in the repository.

2. See the `logs/task5` folder in the repository.

3. See the `logs/task5` folder in the repository.

### Task 6

1. Screenshot of the HAProxy stats page:
    ![](https://i.imgur.com/HrOn6xg.png)

2. This final solution is scalable, which is definitely better than the original one, but there are still a few improvements possible. First of all, only one load balancer is available which means that on the day it fails, the entire system will be down. It could be good idea to set up another load balancer that could take over in that case, so that the system stays alive and nothing will be noticed by the users. It would also be nice to automate the launching (or shutting down) of the webapps as it would quickly get overwhelming for an admin to take care of a fast changing system.
