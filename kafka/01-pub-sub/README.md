# Publisher/Subscriber (Producer/Consumer)

```sh
sh> docker-compose up
```

```sh
sh> docker exec -it kafka /usr/bin/kafka-configs --version
7.2.0-ccs (Commit:b3e5bd063f47e0a6a8dda11d53b84f253ad19491)
```

## Testing

### CMAK 설정

- Add Cluster
  - Cluster Zookeeper Hosts - `zk:2181`
  - [x] Enable ZMX Polling
  - [x] Poll consumer information

### 토픽 생성

```sh
sh> docker exec -it kafka /usr/bin/kafka-topics --bootstrap-server kafka:9092 --create --topic my-topic --partitions 1 --replication-factor 1
Created topic my-topic.

sh> docker exec -it kafka /usr/bin/kafka-topics --bootstrap-server kafka:9092 --list
my-topic
```

### Consume Messages

```sh
sh> docker exec -it kafka /usr/bin/kafka-console-consumer --bootstrap-server kafka:9092 --topic my-topic
# (Waiting...)
```

### Produce Messages

- 새로운 shell 세션에서 열기

```sh
sh> docker exec -it kafka /usr/bin/kafka-console-producer --bootstrap-server kafka:9092 --topic my-topic
> test message
```

### Consumer 세션에서 조회

```sh
test message
```
