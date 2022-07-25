# Errors

```sh
[2022-07-25 06:15:11,601] DEBUG [SocketServer listenerType=ZK_BROKER, nodeId=1] Connection with /192.168.48.4 (channelId=192.168.48.3:9092-192.168.48.4:57020-73) disconnected (org.apache.kafka.common.network.Selector)
java.io.EOFException
  at org.apache.kafka.common.network.NetworkReceive.readFrom(NetworkReceive.java:97)
  at org.apache.kafka.common.network.KafkaChannel.receive(KafkaChannel.java:452)
  at org.apache.kafka.common.network.KafkaChannel.read(KafkaChannel.java:402)
  at org.apache.kafka.common.network.Selector.attemptRead(Selector.java:674)
  at org.apache.kafka.common.network.Selector.pollSelectionKeys(Selector.java:576)
  at org.apache.kafka.common.network.Selector.poll(Selector.java:481)
  at kafka.network.Processor.poll(SocketServer.scala:1144)
  at kafka.network.Processor.run(SocketServer.scala:1047)
  at java.base/java.lang.Thread.run(Thread.java:829)
```
