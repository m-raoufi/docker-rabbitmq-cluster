Docker images to run RabbitMQ cluster. It extends the official image with a rabbitmq-cluster script that does the magic.

# Building

Once you clone the project locally use [captain](https://github.com/m-raoufi/docker-rabbitmq-cluster) to build the image or do with docker:

```
docker build -t afra-repo.jiring.local:8083/rabbitmq-cluster .
```

# Running with docker-stack

If you want to run the cluster on one machine use [docker swarm](https://docs.docker.com/engine/swarm/swarm-tutorial/)

```
docker stack deploy --compose-file docker-stack.yml rabbitmq-cluster
```

By default 3 nodes are started up this way:

```
version: "3.2"
 services:
   master_rabbitmq:
     image: afra-repo.jiring.local:8083/rabbitmq-cluster
     hostname: mastre_rabbitmq
     environment:
       - ERLANG_COOKIE=abcdefg
       - RABBITMQ_DEFAULT_USER=coreswitch
       - RABBITMQ_DEFAULT_PASS=coreswitch
#    volumes:
#        - ~/.docker-conf/rabbitmq/data/:/var/lib/rabbitmq/
#        - ~/.docker-conf/rabbitmq/log/:/var/log/rabbitmq
     ports:
       - target: 5672
         published: 5672
         target: 15672
         published: 15672
         protocol: tcp
         mode: host
     deploy:
         mode: replicated
         replicas: 1
         endpoint_mode: dnsrr
     networks:
       - core_switch
   slave1_rabbitmq:
     image: afra-repo.jiring.local:8083/rabbitmq-cluster
     hostname: slave1_rabbitmq
     links:
       - master_rabbitmq
     environment:
       - ERLANG_COOKIE=abcdefg
       - CLUSTERED=true
       - CLUSTER_WITH=master_rabbitmq
       - ENABLE_RAM=true
     ports:
       - target: 5673
         published: 5672
         target: 15673
         published: 15672
         protocol: tcp
         mode: host
     deploy:
         mode: replicated
         replicas: 1
         endpoint_mode: dnsrr
     networks:
       - core_switch
   slave2_rabbitmq:
     image: afra-repo.jiring.local:8083/rabbitmq-cluster
     hostname: slave2_rabbitmq
     links:
       - master_rabbitmq
       - slave1_rabbitmq
     environment:
       - ERLANG_COOKIE=abcdefg
       - CLUSTERED=true
       - CLUSTER_WITH=master_rabbitmq
     ports:
       - target: 5674
         published: 5672
         protocol: tcp
         mode: host
     deploy:
         mode: replicated
         replicas: 1
         endpoint_mode: dnsrr
     networks:
       - core_switch networks:
  core_switch:
    external: true

```
If needed, additional nodes can be added to this file.

Once cluster is up:
* The management console can be accessed at `http://hostip:15672`
* The connection host should look like this: `hostip:5672,hostip:5673,hostip:5674`

# Credits

* Inspired by https://github.com/harbur/docker-rabbitmq-cluster

