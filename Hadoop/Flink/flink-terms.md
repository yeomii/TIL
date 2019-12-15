# flink terms

* 정리중
* https://ci.apache.org/projects/flink/flink-docs-release-1.9/concepts/glossary.html

### Operator
* 논리적 그래프의 노드.
* 보통 Function 에 의해 특정 연산을 수행한다
* Source 와 Sink 는 데이터 ingestion / egress 를 위한 특별한 operator 종류이다
* 하나 또는 그 이상의 DataStream 을 새로운 DataStream 으로 변형시킨다

### Event Time
* Event 를 발생시키는 곳에서 각 event 가 발생한 시간
* 보통 event 의 각 레코드로부터 event timestamp 값을 얻을 수 있다 
* 현실의 시간과는 무관하게 흘러간다 
    * Stream processor 에서 event time 이 어디까지 진행되었는지 알 수 있는 방법 필요 
    * (Event time + WindowedStream)