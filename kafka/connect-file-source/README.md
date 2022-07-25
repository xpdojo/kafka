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
> curl localhost:8083
{"version":"7.2.0-ccs","commit":"b3e5bd063f47e0a6a8dda11d53b84f253ad19491","kafka_cluster_id":"qD-P28EATQau-eJ8WkQ-tA"}

> curl localhost:8083/connector-plugins
[{"class":"org.apache.kafka.connect.mirror.MirrorCheckpointConnector","type":"source","version":"7.2.0-ccs"},{"class":"org.apache.kafka.connect.mirror.MirrorHeartbeatConnector","type":"source","version":"7.2.0-ccs"},{"class":"org.apache.kafka.connect.mirror.MirrorSourceConnector","type":"source","version":"7.2.0-ccs"}]
```

### Standalone Mode

- [docs](https://docs.confluent.io/home/connect/self-managed/userguide.html#configuring-and-running-workers)

```sh
# -daemon
docker exec connect \
  /usr/bin/connect-standalone \
  # /etc/kafka/connect-standalone.properties \
  /etc/kafka/connect-file-source.properties \
  /etc/kafka/connect-file-sink.properties

[2022-07-25 08:03:43,184] ERROR Stopping due to error (org.apache.kafka.connect.cli.ConnectStandalone)
org.apache.kafka.connect.errors.ConnectException: Unable to initialize REST server
  at org.apache.kafka.connect.runtime.rest.RestServer.initializeServer(RestServer.java:200)
  at org.apache.kafka.connect.cli.ConnectStandalone.main(ConnectStandalone.java:86)
Caused by: java.io.IOException: Failed to bind to 0.0.0.0/0.0.0.0:8083
  at org.eclipse.jetty.server.ServerConnector.openAcceptChannel(ServerConnector.java:349)
  at org.eclipse.jetty.server.ServerConnector.open(ServerConnector.java:310)
  at org.eclipse.jetty.server.AbstractNetworkConnector.doStart(AbstractNetworkConnector.java:80)
  at org.eclipse.jetty.server.ServerConnector.doStart(ServerConnector.java:234)
  at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:73)
  at org.eclipse.jetty.server.Server.doStart(Server.java:401)
  at org.eclipse.jetty.util.component.AbstractLifeCycle.start(AbstractLifeCycle.java:73)
  at org.apache.kafka.connect.runtime.rest.RestServer.initializeServer(RestServer.java:198)
  ... 1 more
Caused by: java.net.BindException: Address already in use
  at java.base/sun.nio.ch.Net.bind0(Native Method)
  at java.base/sun.nio.ch.Net.bind(Net.java:459)
  at java.base/sun.nio.ch.Net.bind(Net.java:448)
  at java.base/sun.nio.ch.ServerSocketChannelImpl.bind(ServerSocketChannelImpl.java:227)
  at java.base/sun.nio.ch.ServerSocketAdaptor.bind(ServerSocketAdaptor.java:80)
  at org.eclipse.jetty.server.ServerConnector.openAcceptChannel(ServerConnector.java:344)
  ... 8 more
```

```sh
docker exec connect cat /etc/kafka/connect-file-source.properties
```

## Distributed Mode

- [docs](https://docs.confluent.io/home/connect/self-managed/userguide.html#distributed-mode)

### 생성된 Connector 확인

```sh
> curl localhost:8083/connectors
[]
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
> curl localhost:8081/config
{"compatibilityLevel":"BACKWARD"}
```
