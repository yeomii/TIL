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
* KeyedStream 에서 사용 가능한 state primitives
    * ValueState<T>
    * ListState<T>
    * ReducingState<T>
    * AggregatingState<IN, OUT>
    * FoldingState<T, ACC>
    * MapState<UK, UV>
* state 를 다루려면 `StateDescriptor` 를 생성해야 한다
* 어떤 state 를 사용하느냐에 따라 사용해야 하는 `StateDescriptor` 도 달라진다
* `State` 는 `RuntimeContext` 를 통해 접근 가능하기 떄문에 rich function 에서만 사용 가능하다

* `ValueState` 를 사용해서 2개씩 평균을 구하는 예제

```kotlin
import org.apache.flink.api.common.functions.RichFlatMapFunction
import org.apache.flink.api.common.state.ValueState
import org.apache.flink.api.common.state.ValueStateDescriptor
import org.apache.flink.api.common.typeinfo.TypeHint
import org.apache.flink.api.common.typeinfo.TypeInformation
import org.apache.flink.api.java.tuple.Tuple2
import org.apache.flink.configuration.Configuration
import org.apache.flink.util.Collector

object StateTestJob {
    class CountWindowAverage(): RichFlatMapFunction<Tuple2<Long, Long>, Tuple2<Long, Long>>() {
        var sum: ValueState<Tuple2<Long, Long>>? = null

        override fun open(parameters: Configuration?) {
            super.open(parameters)
            val descriptor = ValueStateDescriptor(
                "average",
                TypeInformation.of(object : TypeHint<Tuple2<Long, Long>>() {})
            )
            sum = runtimeContext.getState(descriptor)
        }

        override fun flatMap(value: Tuple2<Long, Long>?, out: Collector<Tuple2<Long, Long>>?) {
            value?.let {
                val newSum = Tuple2((sum?.value()?.f0?:0L) + 1, (sum?.value()?.f1?:0L) + value.f1)
                sum?.update(newSum)

                if (newSum.f0 >= 2L) {
                    out?.collect(newSum)
                    sum?.clear()
                }
            }
        }

    }
}

fun main() {
    val env = KafkaConsumerTestJob.getStreamExecutionEnvironment()
    env.parallelism = 1

    env.fromElements(
        Tuple2(1L, 3L), Tuple2(1L, 5L), Tuple2(1L, 7L), Tuple2(1L, 4L), Tuple2(1L, 2L), Tuple2(1L, 1L)
    ).keyBy(0)
        .flatMap(StateTestJob.CountWindowAverage())
        .print()

    env.execute("KoreJob")
}

// output
// (2,8)
//(2,11)
// (2,3)
```

### State Time-To-Live (TTL)
* 어떤 타입의 keyed state 에도 TTL 을 적용할 수 있다
* 모든 collectio type 의 state 는 엔트리별 ttl 을 지원한다
* TTL 을 적용하기 위해서는 `StateTtlConfig` 를 생성해서 descriptor 에 넘겨주면 된다

* ttl config 사용 예

```kotlin
val ttlConfig = StateTtlConfig
    .newBuilder(Time.seconds(1))
    .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
    .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
    .build()

val descriptor = ValueStateDescriptor(
    "average",
    TypeInformation.of(object : TypeHint<Tuple2<Long, Long>>() {})
)
descriptor.enableTimeToLive(ttlConfig)
```

* state backend 에 저장할 때는 값과 함께 최근 수정 시간도 기록하기 때문에 state storage 사용량이 증가할 수 있다
* 현재 processing time 을 참조하는 ttl 만 지원된다
* ttl 없이 저장된 state 를 ttl 설정이 활성화 된 채로 복원하려고 하면 호환성 에러가 발생한다
* ttl 설정은 체크포인트나 세이브포인트의 일부가 아니고, 현재 동작하는 잡을 플링크가 다루는 방식이라고 보는 것이 좋다
* 현재 ttl 이 적용된 map state 는 serializer 가 null 값을 지원할 때에만 null 을 사용할 수 있다

* 만료된 상태의 정리
    * 기본적으로는 만료된 값은 명시적으로 읽힐 때에만 지워질 수 있다
    * 즉 만료된 상태들이 읽히지 않은 상태로 계속 state 에 지워지지 않은채로 쌓일 수 있다는 것

### Using Managed Operator State
* `CheckpointedFunction` 
    * 다른 재분배 스킴을 가진 non-keyed state 를 이 인터페이스로 접근할 수 있다
    * 인터페이스는 아래 두 인터페이스를 구현해야 한다,

```java
void snapshotState(FunctionSnapshotContext context) throws Exception;
void initializeState(FunctionInitializationContext context) throws Exception;
```

* 체크포인트가 수행되면 `snapshotState()` 가 호출된다
* 반대로 사용자가 정의한 함수가 초기화 될 때 마다 `initializestate()` 가 호출된다

* list 스타일의 managed operator state 가 지원된다
* state 는 serializable 한 객체의 리스트여야 하며, 객체끼리는 서로 의존성이 없어야 한다 (rescaling 시 재분배 하기 좋기 때문에)
* state 접근 방법에 따라 아래 재분배 스킴이 정의된다
    * 각 operator 는 state element 의 리스트를 반환
    * 전체 state 는 모든 리스트를 concat 한 것
    * even-split 재분배
        * 복구시 전체 리스트를 parallel operator 수 만큼 고르게 쪼개서 각 operator instance 에 sublist 하나씩 분배한다
    * union 재분배
        * 복구시 전체 리스트를 모든 operator instance 에 전달한다


* CheckpointedFunction 을 사용하는 stateful SinkFunction 예제
    * `State` 객체는 operator 의 스냅샷 상태를 저장하거나, 재시작시 복구한 state 스냅샷을 참조할 때 사용된다
    * initializeState 에서 `context?.isRestored` 를 확인하여 failure 로부터 복구되었는지 확인한 후, 복구된 state 스냅샷으로부터 operator 의 상태를 복원한다
```kotiln 
class BufferingSink(val threshould: Int, val bufferedElements: MutableList<String>)
    : CheckpointedFunction, SinkFunction<String> {
    var checkpointedState: ListState<String>? = null

    override fun invoke(value: String?, context: SinkFunction.Context<*>?) {
        super.invoke(value, context)
        value?.let {
            bufferedElements.add(value)
            if (bufferedElements.size >= threshould) {
                println(Date().toString() + " - " + bufferedElements.joinToString(" "))
                bufferedElements.clear()
            }
        }
    }

    override fun initializeState(context: FunctionInitializationContext?) {
        val descriptor = ListStateDescriptor(
            "buffered-elements",
            TypeInformation.of(String::class.java)
        )
        checkpointedState = context?.operatorStateStore?.getListState(descriptor)
        if (context?.isRestored == true)
            checkpointedState?.get()?.forEach { x -> bufferedElements.add(x) }
    }

    override fun snapshotState(context: FunctionSnapshotContext?) {
        checkpointedState?.clear()
        checkpointedState?.addAll(bufferedElements)
    }
}
```


* Stateful Source Functions
    * stateful 소스는 장애 복구시 `exactly-once` 시맨틱을 위해 checkpoint 용 lock 을 획득해서 처리할 필요가 있다
```kotlin
class CounterSource: ListCheckpointed<Long>, RichParallelSourceFunction<Long>()  {
    private var offset: Long = 0L
    private var isRunning: Boolean = true

    override fun cancel() {
        isRunning = false
    }

    override fun run(ctx: SourceFunction.SourceContext<Long>?) {
        while (isRunning) {
            ctx?.checkpointLock?.let {
                synchronized(it) {
                    ctx.collect(offset)
                    offset += 1
                }
            }
        }
    }

    override fun restoreState(state: MutableList<Long>?) {
        offset = state?.max()?:0L
    }

    override fun snapshotState(checkpointId: Long, timestamp: Long): MutableList<Long> {
        return mutableListOf(offset)
    }
}
```


* 어떤 operator 는 checkpoint 가 완료될 때 외부에 알려야 할 필요가 있을 수 있다
    * 이를 위해 flink 에서는 `org.apache.flink.runtime.state.CheckpointListener` 를 제공한다

