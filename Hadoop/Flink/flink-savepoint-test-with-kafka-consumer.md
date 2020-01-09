# Flink Savepoint test with kafka consumer

## 배경지식
### state
* https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/stream/state/state.html
* flink function 이나 operator 가 처리했던 element 들에 대한 데이터를 가지고 있어야 하는 경우 (stateful 한 경우) flink 에서 제공하는 state 를 통해 해당 데이터를 저장할 수 있다
### checkpointing
* https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/stream/state/checkpointing.html
* state 를 주기적으로 저장하고, 예상하지 못한 fail 로 부터 잡이 재시작 될 때 해당 state 를 복원할 수 있도록 하는 방법을 제공한다
### savepoint
* https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/savepoints.html
* checkpoint 의 동작방식과 유사하게 state 를 저장하고 저장된 savepoint 로 부터 state 를 저장하는 방법
* 사용자가 직접 state 저장을 트리거링하고, 저장된 savepoint 로부터의 복원도 사용자가 직접 지정해야 한다는 것이 checkpoint 와의 차이점

## 문제 개요
* checkpointing 만으로 state 를 복구하는 것은 flink 의 restart strategy 에 의해 재시작이 되는 경우만 가능하다
  * -> 예상하지 못한 fail 로 재시작하는 경우만 복구 가능, 사용자가 의도한 stop-and-resume 의 경우 복구 불가 (https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/savepoints.html#what-is-a-savepoint-how-is-a-savepoint-different-from-a-checkpoint)
* 사용자가 의도한 stop-and-resume 케이스에서의 state 복원은 savepoint 를 사용해야 함

## 테스트 코드
* [Flink Kafka Consumer Offset Test](./flink-kafka-consumer-offset-test.md) 의 컨슈머와 프로듀서 사용

## 테스트 준비
* 테스트 전에 아래 커맨드로 로컬 클러스터 시작해주기 (1 cluster, 2 task-executor 준비)
```sh
$ ./bin/flink-1.7.2/bin/start-cluster.sh
$ ./bin/flink-1.7.2/bin/taskmanager.sh start
```
* localhost:8081 에서 대시보드 확인
* KafkaProducerTestJob 을 실행해서 테스트 토픽에 메시지 생성

## 테스트 방법

1. 아래 커맨드로 KafkaConsumerTest 잡을 cli 로 시작한다
  a. ~/bin/flink-1.7.2/bin/flink run -c com.test.KafkaConsumerTestJobKt ./build/libs/flink-kotlin-example.jar
2. savepoint 를 만드려면 job id 확인 후 아래와 같은 커맨드를 실행해준다
  a. ~/bin/flink-1.7.2/bin/flink savepoint :jobId :savepointRootPath
  b. ex) ~/bin/flink-1.7.2/bin/flink savepoint ddfa6aba18de29ad271fc46825756781 ~/test/save
3. local flink web ui 대시보드에서 진행중인 잡 캔슬하기
4. 아래 커맨드로 KafkaConsumerTest 잡을 savepoint 를 지정해서 재실행한다
  a. ~/bin/flink-1.7.2/bin/flink run -s :savepointPath -c com.test.KafkaConsumerTestJobKt ./build/libs/flink-kotlin-example.jar
  b. ex) ~/bin/flink-1.7.2/bin/flink run -s ~/test/save/savepoint-ddfa6a-dd2cab343d46 -c com.test.KafkaConsumerTestJobKt ./build/libs/flink-kotlin-example.jar
5. local flink web ui 대시보드 > taskmanagers > stdout 에서 로그 확인하기

## 테스트 결과
* savepoint 로 Consumer offset 복원하여 재시작
* 일부 메시지기 중복처리된다

## Flink Consumer 의 offset 복원 관련 로그
* state 복구 과정 없이 consumer 를 띄울 경우 
```
2020-01-07 18:38:54,137 INFO  org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumerBase  - No restore state for FlinkKafkaConsumer.
...
2020-01-07 18:38:54,471 INFO  org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumerBase  - Consumer subtask 0 will start reading the following 4 partitions from the committed group offsets in Kafka: [KafkaTopicPartition{topic='test-topic', partition=6}, KafkaTopicPartition{topic='test-topic', partition=4}, KafkaTopicPartition{topic='test-topic', partition=2}, KafkaTopicPartition{topic='test-topic', partition=0}]
```
* state 로부터 offset 을 복원하여 consumer 를 띄울 경우 
```
2020-01-07 18:50:28,665 INFO  org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumerBase  - Setting restore state in the FlinkKafkaConsumer: {KafkaTopicPartition{topic='test-topic', partition=0}=487, KafkaTopicPartition{topic='test-topic', partition=1}=463, KafkaTopicPartition{topic='test-topic', partition=2}=464, KafkaTopicPartition{topic='test-topic', partition=3}=464, KafkaTopicPartition{topic='test-topic', partition=4}=463, KafkaTopicPartition{topic='test-topic', partition=5}=455, KafkaTopicPartition{topic='test-topic', partition=6}=452, KafkaTopicPartition{topic='test-topic', partition=7}=448}
...
2020-01-07 18:50:28,862 INFO  org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumerBase
```