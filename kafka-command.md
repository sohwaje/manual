# Kafka 설치 후 테스트
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

- 토픽 상세 조회
    + --bootstrap-server KF01:9092,KF02:9092로 나열할 수 있음
```
./kafka-topics.sh --bootstrap-server localhost:9092  --topic TOPIC --describe
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

# Kafka JMX Monitoring
- jar 파일과 모니터링 메트릭 설정 파일 다운로드
```
sudo wget -P /usr/local/kafka/bin https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.12.0/jmx_prometheus_javaagent-0.12.0.jar
sudo wget -P /usr/local/kafka/bin https://raw.githubusercontent.com/prometheus/jmx_exporter/master/example_configs/kafka-2_0_0.yml
```

- kafka-2_0_0.yml에 아래 설정 추가
```
vim /usr/local/kafka/bin/kafka-2_0_0.yml
- pattern : kafka.producer<type=producer-metrics, client-id=(.+)><>(.+):\w*
  name: kafka_producer_$2
- pattern : kafka.consumer<type=consumer-metrics, client-id=(.+)><>(.+):\w*
  name: kafka_consumer_$2
- pattern : kafka.consumer<type=consumer-fetch-manager-metrics, client-id=(.+)><>(.+):\w*
  name: kafka_consumer_$2
```

```
sudo vi /usr/local/kafka/bin/kafka-server-start.sh
export KAFKA_OPTS="-javaagent:$base_dir/../bin/jmx_prometheus_javaagent-0.12.0.jar=7071:$base_dir/../bin/kafka-2_0_0.yml"
```

```
subo vi /usr/local/kafka/bin/kafka-console-consumer.sh
export KAFKA_OPTS="-javaagent:$base_dir/../bin/jmx_prometheus_javaagent-0.12.0.jar=7071:$base_dir/../bin/kafka-2_0_0.yml"
```

```
subo vi /usr/local/kafka/bin/kafka-console-producer.sh
export KAFKA_OPTS="-javaagent:$base_dir/../bin/jmx_prometheus_javaagent-0.12.0.jar=7071:$base_dir/../bin/kafka-2_0_0.yml"
```
- 카프카 재시작
```
sudo systemctl restart kafka.service
```

- 프로메테우스 설정
```
sudo vi prometheus.yml
...
  - job_name: 'kafka'
    static_configs:
      - targets: ['10.7.1.11:7071','10.7.1.14:7071']
...
```
# Troubleshooting
> kafka.common.InconsistentClusterIdException: The Cluster ID _jWF8TmzSuS1px1Po23QUA doesn't match stored clusterId Some(qrDaiotkT5iQBG55lIUqvA) in meta.properties. The broker is trying to join the wrong cluster. Configured zookeeper.connect may be wrong.
        at kafka.server.KafkaServer.startup(KafkaServer.scala:223)
        at kafka.Kafka$.main(Kafka.scala:109)
        at kafka.Kafka.main(Kafka.scala)

```
cd $KAFKA_LOG_DIR/
rm -f meta.properties
```
- 카프카에서 주키퍼 삭제 시 아래와 같이 ***rmr*** 명령어를 찾을 수 없다는 에러가 뜬다. rmr대신 ***deleteall*** 을 사용해야 한다.
> Command not found: Command not found rmr
- 주키퍼 접속
```
kafka01]# ./zookeeper-shell.sh zk01:2181,zk02:2181,zk03:2181
Welcome to ZooKeeper!
JLine support is disabled

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
```

- 삭제할 토픽 찾기
```
ls /brokers/topics
[79i6bx, __consumer_offsets]
```

- 토픽 삭제
```
deleteall /kafka01_znode/brokers/topics/79i6bx
```
