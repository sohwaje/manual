### topic 
- 생성(topic 이름은 test)
```
./kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
```
- 목록 확인
```
./kafka-topics.sh --list --bootstrap-server localhost:9092

79i6bx
__consumer_offsets
test
```

### producer
- 메시지 보내기
```
./kafka-console-producer.sh --broker-list localhost:9092 --topic test
```

### consumer
- 메시지 받기
```
./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
```


### Troubleshooting
> kafka.common.InconsistentClusterIdException: The Cluster ID _jWF8TmzSuS1px1Po23QUA doesn't match stored clusterId Some(qrDaiotkT5iQBG55lIUqvA) in meta.properties. The broker is trying to join the wrong cluster. Configured zookeeper.connect may be wrong.
        at kafka.server.KafkaServer.startup(KafkaServer.scala:223)
        at kafka.Kafka$.main(Kafka.scala:109)
        at kafka.Kafka.main(Kafka.scala)

```
cd $KAFKA_LOG_DIR/
rm -f meta.properties
```