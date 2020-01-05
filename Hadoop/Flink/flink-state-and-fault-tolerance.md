# Flink State & Fault tolerance

## Working with State
* https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/stream/state/state.html

### Basic 2 states
* Keyed State
* Operator State

### Keyed State
* KeyedStream 함수나 스트림에서만 사용할 수 있는 key 와 연관된 state
* Operator state 가 key 에 따라 파티션되고 샤딩된 거라고 생각할 수 있다
* 논리적으로 각 keyed state 는 고유한 `<parallel-operator-instance, key>` 결합 하나당 하나가 있는것으로 생각할 수 있다. 왜냐하면 하나의 key 는 keyed operator 의 parallel instance 하나에만 속하기 때문이다. 
* Keyed State 는 Key Group 이라고 불리는 것을 구성할 수 있다.
* Key Group 은 플링크가 Keyed State 를 재배포할 수 있는 아토믹한 단위이다.
* maximum parallelism 만큼의 key group 이 있다고 보면 된다.

## Operator State
* 각 operator state 는 하나의 `parallel-operator-instance` 에 속한다.
* kafka connector 가 flink 의 operator state 를 사용하는 좋은 예다.
* kafka consumer 의 각 parallel instance 들은 토픽 파티션과 offset 정보의 map 을 operator state fh emfrhdlTek
* operator state 인터페이스는 parallelism 이 바뀌면서 parallel operator instance 가 재배포될 때 어떻게 state 를 재배포할 것인지 정의할 수 있게 해준다.

### 2 forms of state
* Raw State
    * state 가 자기 고유의 자료구조를 가진다.
    * 체크포인트시에는 단순히 state 의 byte 열을 기록한다.
* Managed State
    * internal hash table 이나 RocksDB 와 같이 flink runtime 에 의헤 제어되는 자료구조로 표현된다. 
    * `ValueState`, `ListState` 등...
    * Flink 런타임은 이 state 를 인코딩해서 checkpoint 에 쓴다
* 모든 datastream 은 managed state 를 사용할 수 있지만 raw state 는 operator 를 구현해야만 사용할 수 있다.
* managed state 를 사용하는 것이 권장되며, parallelism 값이 변경될 때,  managed state 의 경우 flink 가 알아서 state 재분배를 해주기 때문에 메모리 관리 측면에서도 좋다.


### Managed Keyed State 사용하기
