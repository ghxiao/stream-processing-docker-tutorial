# Lab 8: Setup Flink and Kafka Clusters using Docker

## Flink

Reference: <https://hub.docker.com/_/flink>

### Running a JobManager or a TaskManager

You can run a JobManager (master).

```console
$ docker run --name flink_jobmanager -d -t flink jobmanager
```

NB: if you have already the container created, the above command will lead to an error like below:
`docker: Error response from daemon: Conflict. The container name "/flink_jobmanager" is already in use by container "32523f311c747cf6b0c0744d9edda2788af2213da189cd7926f60a4192073ce8". You have to remove (or rename) that container to be able to reuse that name.`
In this case, run 
```console
docker start flink_jobmanager
```

You can also run a TaskManager (worker). Notice that workers need to register with the JobManager directly or via ZooKeeper so the master starts to send them tasks to execute.

```console
$ docker run --name flink_taskmanager -d -t flink taskmanager
```

### Running a cluster using Docker Compose

With Docker Compose you can create a Flink cluster:

```yml
version: "2.1"
services:
  jobmanager:
    image: ${FLINK_DOCKER_IMAGE_NAME:-flink}
    expose:
      - "6123"
    ports:
      - "8081:8081"
    command: jobmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager

  taskmanager:
    image: ${FLINK_DOCKER_IMAGE_NAME:-flink}
    expose:
      - "6121"
      - "6122"
    depends_on:
      - jobmanager
    command: taskmanager
    links:
      - "jobmanager:jobmanager"
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
```

and just run `docker-compose -f docker-compose-flink.yml up --no-recreate`.

Scale the cluster up or down to *N* TaskManagers:

```console
docker-compose -f docker-compose-flink.yml up --scale taskmanager=<N> --no-recreate
```

E.g.,

```console
docker-compose -f docker-compose-flink.yml up --scale taskmanager=3 --no-recreate
```

### Submit a job

#### From the command line

```console
$ flink run  -c io.github.streamingwithflink.chapter1.AverageSensorReadings examples-scala_2.12-1.0.jar
```

#### From browser

- open <http://localhost:8081/#/submit>
- upload the jar file `examples-scala_2.12-1.0.jar`
- specify the Entry Class io.github.streamingwithflink.chapter1.AverageSensorReadings
- Submit

## Kafka

### Running a Kafka cluster using Docker Compose

<https://hub.docker.com/r/bitnami/kafka/>

```
$ docker-compose -f docker-compose-kafka.yml up --no-recreate
```

### Or use the Confluent Docker setup

https://docs.confluent.io/current/quickstart/ce-docker-quickstart.html

## Reference:

- Docker: https://qconsf2017intro.container.training/