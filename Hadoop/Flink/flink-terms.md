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

### Watermark 
* Data stream 을 따라서 흘러가며, timestamp 값을 전달
* Watermark(t) 는 스트림 내의 event time 이 t 까지 진행되었고, `W(t)` 이후에는 t 보다 먼저 발생한 이벤트가 더 이상 없어야 함을 의미
    * 늦게 들어오는 이벤트에 대한 처리는 operator 마다 다름
* operator 가 흘러온 watermark 를 받으면, 내부 event time clock 을 해당 워터마크의 값으로 진행시킴