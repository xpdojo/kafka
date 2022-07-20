# Apache Kafka

```sh
sh> docker-compose up
```

```sh
sh> docker exec -it kafka-1 /usr/bin/kafka-configs --version
7.2.0-ccs (Commit:b3e5bd063f47e0a6a8dda11d53b84f253ad19491)

sh> docker exec -it kafka-2 /usr/bin/kafka-configs --version
7.2.0-ccs (Commit:b3e5bd063f47e0a6a8dda11d53b84f253ad19491)
```

## CLI

### 토픽 생성

```sh
sh> docker exec -it kafka-2 /usr/bin/kafka-topics --bootstrap-server localhost:9092 --create  --replication-factor 1 --topic my-topic
sh> docker exec -it kafka-2 /usr/bin/kafka-topics --bootstrap-server localhost:9092 --list
my-topic
```
