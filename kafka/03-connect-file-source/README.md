# CDC 데이터 동기화 (테스트중)

```sh
> docker exec -it kafka bash
```

```sh
> ls -al /etc/kafka
-rw-rw-r--. 1 appuser root     906 Jul  5 04:18 connect-console-sink.properties
-rw-rw-r--. 1 appuser root     909 Jul  5 04:18 connect-console-source.properties
-rw-rw-r--. 1 appuser root    5489 Jul  5 04:18 connect-distributed.properties
-rw-rw-r--. 1 appuser root     883 Jul  5 04:18 connect-file-sink.properties
-rw-rw-r--. 1 appuser root     881 Jul  5 04:18 connect-file-source.properties
-rw-rw-r--. 1 appuser root    2103 Jul  5 04:18 connect-log4j.properties
-rw-rw-r--. 1 appuser root    2540 Jul  5 04:18 connect-mirror-maker.properties
-rw-rw-r--. 1 appuser root    2276 Jul  5 04:18 connect-standalone.properties
-rw-rw-r--. 1 appuser root    1221 Jul  5 04:18 consumer.properties
-rw-r--r--. 1 appuser appuser  365 Jul 24 06:41 kafka.properties
drwxrwxr-x. 1 appuser root     128 Jul  5 10:12 kraft
-rw-rw-r--. 1 appuser root     541 Jul 24 06:41 log4j.properties
-rw-rw-r--. 1 appuser root    2065 Jul  5 04:18 producer.properties
drwxrwxr-x. 1 appuser root       0 Jul  5 10:12 secrets
-rw-rw-r--. 1 appuser root    7534 Jul  5 04:18 server.properties
-rw-rw-r--. 1 appuser root     251 Jul 24 06:41 tools-log4j.properties
-rw-rw-r--. 1 appuser root    1169 Jul  5 04:18 trogdor.conf
-rw-rw-r--. 1 appuser root    1209 Jul  5 04:18 zookeeper.properties
```

## Kafka Connect

```sh
# Kafka Connect REST Server
curl localhost:8083 | jq
{
  "version": "7.2.0-ccs",
  "commit": "b3e5bd063f47e0a6a8dda11d53b84f253ad19491",
  "kafka_cluster_id": "XG1_JwFJRhqAwMG-dh_Tpg"
}
```

- 생성된 Connector 조회

```sh
curl localhost:8083/connectors
[]
```

- 설치된 Connector Plugin 조회

```sh
curl localhost:8083/connector-plugins | jq '.[] | .class'

"org.apache.kafka.connect.file.FileStreamSinkConnector"
"org.apache.kafka.connect.file.FileStreamSourceConnector"
"org.apache.kafka.connect.mirror.MirrorCheckpointConnector"
"org.apache.kafka.connect.mirror.MirrorHeartbeatConnector"
"org.apache.kafka.connect.mirror.MirrorSourceConnector"
```

- File Source Connector 생성

```sh
curl -i -X PUT -H  "Content-Type:application/json" \
  http://localhost:8083/connectors/local-file-source/config \
  -d @./connect/local-file-source.json

# output
HTTP/1.1 201 Created
Date: Tue, 26 Jul 2022 09:18:27 GMT
Location: http://localhost:8083/connectors/local-file-source
Content-Type: application/json
Content-Length: 231
Server: Jetty(9.4.44.v20210927)

{"name":"local-file-source","config":{"connector.class":"org.apache.kafka.connect.file.FileStreamSourceConnector","tasks.max":"1","file":"/tmp/test.txt","topic":"connect-test","name":"local-file-source"},"tasks":[],"type":"source"}
```

- File Sink Connector 생성

```sh
curl -i -X PUT -H  "Content-Type:application/json" \
  http://localhost:8083/connectors/local-file-sink/config \
  -d @./connect/local-file-sink.json

# output
HTTP/1.1 200 OK
Date: Tue, 26 Jul 2022 09:20:44 GMT
Content-Type: application/json
Content-Length: 269
Server: Jetty(9.4.44.v20210927)

{"name":"local-file-sink","config":{"connector.class":"org.apache.kafka.connect.file.FileStreamSinkConnector","tasks.max":"1","file":"/tmp/test.sink.txt","topics":"connect-test","name":"local-file-sink"},"tasks":[{"connector":"local-file-sink","task":0}],"type":"sink"}
```

- 생성된 Connector 조회

```sh
curl localhost:8083/connectors | jq
[
  "local-file-source",
  "local-file-sink"
]
```

- 파일 내용을 변경해서 정상적으로 동작하는지 테스트한다.

```sh
docker exec connect cat /tmp/test.txt 
# Hello Kafka!
docker exec connect cat /tmp/test.sink.txt
# Hello Kafka!

docker exec connect sh -c "echo $(date) >> /tmp/test.txt"
docker exec connect cat /tmp/test.txt
# Hello Kafka!
# Tue Jul 26 06:24:30 PM KST 2022
docker exec connect cat /tmp/test.sink.txt
# Hello Kafka!
# Tue Jul 26 06:24:30 PM KST 2022
```

### Standalone Mode

- [docs](https://docs.confluent.io/home/connect/self-managed/userguide.html#configuring-and-running-workers)

```sh
docker exec connect cat /etc/kafka/connect-standalone.properties
```

### Distributed Mode

- [docs](https://docs.confluent.io/home/connect/self-managed/userguide.html#distributed-mode)

```sh
docker exec connect cat /etc/kafka/connect-distributed.properties
```

## Schema Registry: Apache Avro

### Schema Registry 호환성 레벨

- `BACKWARD` : **default**. Consumer가 Producer에게 메시지를 보낼 때 동일한 버전과 **하나 이래의** 하위 버전 스키마를 읽을 수 있다.
- `BACKWARD_TRANSITIVE` : Consumer가 Producer에게 메시지를 보낼 때 동일한 버전과 **모든** 하위 버전 스키마를 읽을 수 있다.
- `FORWARD` : Producer가 Consumer에게 메시지를 보낼 때 동일한 버전과 **하나 이래의** 하위 버전 스키마를 읽을 수 있다.
- `FORWARD_TRANSITIVE` : Producer가 Consumer에게 메시지를 보낼 때 동일한 버전과 **모든** 하위 버전 스키마를 읽을 수 있다.
- `FULL` : Producer, Consumer 양측에서 호환. 동일한 버전과 **하나 이래의** 하위 버전 스키마를 읽을 수 있다.
- `FULL_TRANSITIVE` : Producer, Consumer 양측에서 호환. 동일한 버전과 **모든** 하위 버전 스키마를 읽을 수 있다.

```sh
curl localhost:8081/config | jq

{
  "compatibilityLevel": "BACKWARD"
}
```
