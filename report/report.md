# AIT Lab 04

### Introduction
In this lab, we are going to get familiar with dynamic scaling with Docker. It is based on the load balancing lab.

### Task 0
[M1]: No, the current solution would not suit for a production environment. The main problems are the manual launching of the containers. 

[M2]: We would have to modify the configuration file so that a third web app container will be launched along with the first two. More precisely the Docker image will be instancied a third time to create the third container.

[M3]: . 

[M4]: We could let the load balancer determine the number of web application nodes to launch depending on the incoming traffic.

[M5]: .

[M6]: To add more web server nodes, we would have to add more lines like the ones beginning with sed. It is not dynamic as we would have to do that manually for each web server nodes we want to add. A solution would be to give the number of webapp to create and make it create them dynamically?

*Deliverables:*

1. Screenshot of the stats page of HAProxy at this point:

    ![](https://i.imgur.com/h86NYuH.png)
	![Image](/images/task1_haproxy_stats.png)
    
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
    Consul.

### Task 3

1. See the `logs/task3` folder in the repository.

2. See the `logs/task3` folder in the repository.

### Task 4

1. .

2. .

3. See the `logs/task4` folder in the repository.

4. .

### Task 5

1. See the `logs/task5` folder in the repository.

2. See the `logs/task5` folder in the repository.

3. See the `logs/task5` folder in the repository.

### Task 6

1. Screenshot of the HAProxy stats page:
    ![](https://i.imgur.com/HrOn6xg.png)

2. .
