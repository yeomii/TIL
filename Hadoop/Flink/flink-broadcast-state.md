# Flink Broadcast State

* https://ci.apache.org/projects/flink/flink-docs-stable/dev/stream/state/broadcast_state.html

## Provided APIs
전체적인 기능을 설명하기 전에 예제를 먼저 살펴보자. 서로 다른 색과 형태를 가진 아이템들이 입력 스트림으로 들어온다고 할 때, 특정 패턴을 따르는 같은 색의 아이템 쌍을 찾는 문제 상황을 풀어볼 것이다. 예를 들어 사각형 다음에 삼각형이 오는 경우 등의 패턴을 정의할 수 있으며, 찾고자 하는 패턴은 시간에 따라 변화할 수 있다.\
이 예제에서 첫 번째 스트림은 Color 와 Shape 특성을 갖는 Item 요소를 입력으로 갖고, 다른 하나는 찾고자 하는 패턴을 정의하는 Rules 요소를 입력으로 갖는다.\
Items 스트림에 대해서는 Color 로 스트림 키를 정의하며 아래와 같이 표현할 수 있다. 이는 같은 색의 아이템은 같은 물리노드에서 처리됨을 보장한다.\
```
// key the items by color
KeyedStream<Item, Color> colorPartitionedStream = itemStream
                        .keyBy(new KeySelector<Item, Color>(){...});
```
Rules 를 포함하는 스트림은 하류(downstream)의 태스크로 전파(broadcast) 되어야 하고, 하류 태스크들은 rules 내용을 로컬에 저장하여 Item 을 처리할 때 사용할 수 있도록 해야한다. 아래 코드스니펫은 rules 스트림을 브로드캐스팅하고, MapStateDescriptor 를 제공하여 rules 가 저장될 broadcast state 를 생성한다.\
```
// a map descriptor to store the name of the rule (string) and the rule itself.
MapStateDescriptor<String, Rule> ruleStateDescriptor = new MapStateDescriptor<>(
			"RulesBroadcastState",
			BasicTypeInfo.STRING_TYPE_INFO,
			TypeInformation.of(new TypeHint<Rule>() {}));
		
// broadcast the rules and create the broadcast state
BroadcastStream<Rule> ruleBroadcastStream = ruleStream
                        .broadcast(ruleStateDescriptor);
```
마지막으로, Rules를 Item 스트림의 입력 요소에 대해 계산하기 위해서 아래 두가지가 필요하다.\
1. 두 개의 스트림 연결
2. match detecting 로직을 정의

BroadcastStream 과 일반 (keyed or non-keyed) 스트림을 연결하는 것은 non-broadcast 스트림에 대하여 `BroadcastStream` 을 인자로 해서 `connect()` 함수를 호출하면 된다. 호출 함수의 리턴타입은 `BroadcastConnectedStream` 이며, 특별한 타입을 갖는 `CoProcessFunction` 를 정의하여 `process()` 함수를 호출할 수 있다. 여기서 `CoProcessFunction` 에 우리의 매칭 로직을 정의하면 된다. Function 의 정확한 타입은 다음과 같다.\
- keyed stream 일 경우 - `KeyedBroadcastProcessFunction`
- non-keyed stream 일 경우 - `BroadcastProcessFunction`

```
DataStream<String> output = colorPartitionedStream
                 .connect(ruleBroadcastStream)
                 .process(
                     
                     // type arguments in our KeyedBroadcastProcessFunction represent: 
                     //   1. the key of the keyed stream
                     //   2. the type of elements in the non-broadcast side
                     //   3. the type of elements in the broadcast side
                     //   4. the type of the result, here a string
                     
                     new KeyedBroadcastProcessFunction<Color, Item, Rule, String>() {
                         // my matching logic
                     }
                 );
```

## BroadcastProcessFunction and KeyedBroadcastProcessFunction
`CoProcessFunction` 의 경우 두 가지의 process 함수를 구현해주어야 한다. `processBroadcastElement()` 에서는 broadcast 스트림으로부터 들어오는 요소를 처리해야하고, `processElement()` 에서는 non-broadcast 스트림에서 들어오는 요소를 처리해주어야 한다. 아래는 함수들의 시그니쳐이다.\
```
// case of non-keyed stream
public abstract class BroadcastProcessFunction<IN1, IN2, OUT> extends BaseBroadcastProcessFunction {

    public abstract void processElement(IN1 value, ReadOnlyContext ctx, Collector<OUT> out) throws Exception;

    public abstract void processBroadcastElement(IN2 value, Context ctx, Collector<OUT> out) throws Exception;
}

// case of keyed-stream
public abstract class KeyedBroadcastProcessFunction<KS, IN1, IN2, OUT> {

    public abstract void processElement(IN1 value, ReadOnlyContext ctx, Collector<OUT> out) throws Exception;

    public abstract void processBroadcastElement(IN2 value, Context ctx, Collector<OUT> out) throws Exception;

    public void onTimer(long timestamp, OnTimerContext ctx, Collector<OUT> out) throws Exception;
}
```

`processElement` 와 `processBroadcastElement` 는 제공되는 context 가 다르다는 차이점이 있다. non-broadcast 를 처리하는 함수는 `ReadOnlyContext` 를 참조하고, broadcast 요소를 처리하는 함수는 `Context` 를 참조한다. (read-write 접근 가능)\
두 컨텍스트(ctx) 모두 아래 내용을 제공한다.
1.  broadcast state (`ctx.getBroadcastState(MapStateDescriptor<K, V> stateDescriptor)`) 에 대한 접근을 제공한다.
2. 요소의 timestamp 에 대한 쿼리 (`ctx.timestamp()`) 를 허용한다.
3. 현재의 watermark (`ctx.currentWatermark()`) 를 가져올 수 있다.
4. 현재의 처리시간 (`ctx.currentProcessingTime()`) 을 가져올 수 있다.
5. side-outputs (`ctx.output(OutputTag<X> outputTag, X value)`) 으로 요소를 전달할 수 있다.

`getBroadcastState()` 로 가져올 수 있는 `stateDescriptor` 는 `.broadcast(ruleStateDescriptor)` 에 정의된 것과 동일해야 한다.

broadcast 쪽에서는 읽기쓰기 접근이 가능한 반면, non-broadcast 쪽에서는 읽기전용 접근만 가능한 이유는 Flink 에서 태스크 사이 (cross-task)의 통신을 허용하지 않기 떄문이다. 때문에 operator 의 모든 병렬 인스턴스에서 Broadcast State 의 내용이 동일함을 보장하려면 broadcast 쪽에서만 읽기쓰기 접근을 허용해야한다. 이 규칙을 무시한다면 state 의 일관성이 보장되지 않을것이다.

따라서 주의할 것은 `processBroadcastElement()` 에 구현되는 로직은 모든 병렬 인스턴스에 걸쳐 동일하게 deterministic 한 동작이어야 한다.

`KeyedBroadcastProcessFunction` 가 keyed stream 상에서 동작하기 때문에 `BroadcastProcessFunction` 에서는 불가능한 다음과 같은 기능을 가진다.
1. `ReadOnlyContext` 를 통해 Flink 의 타이머 서비스에 접근할 수 있고, processing 시간에 대한 타이머 이벤트를 등록할 수 있다.
2. `processBroadcastElement()` 메서드의 `Context` 는 ` applyToKeyedState(StateDescriptor<S, VS> stateDescriptor, KeyedStateFunction<KS, S> function)` 메서드를 제공하여 해당 `stateDescriptor` 와 관련되 모든키의 모든 state에 적용되는 `KeyedStateFunction` 을 등록할 수 있게 한다.

예제 문제상황에 맞는 KeyedBroadcastProcessFunction 은 다음과 같다
```
new KeyedBroadcastProcessFunction<Color, Item, Rule, String>() {

    // store partial matches, i.e. first elements of the pair waiting for their second element
    // we keep a list as we may have many first elements waiting
    private final MapStateDescriptor<String, List<Item>> mapStateDesc =
        new MapStateDescriptor<>(
            "items",
            BasicTypeInfo.STRING_TYPE_INFO,
            new ListTypeInfo<>(Item.class));

    // identical to our ruleStateDescriptor above
    private final MapStateDescriptor<String, Rule> ruleStateDescriptor = 
        new MapStateDescriptor<>(
            "RulesBroadcastState",
            BasicTypeInfo.STRING_TYPE_INFO,
            TypeInformation.of(new TypeHint<Rule>() {}));

    @Override
    public void processBroadcastElement(Rule value,
                                        Context ctx,
                                        Collector<String> out) throws Exception {
        ctx.getBroadcastState(ruleStateDescriptor).put(value.name, value);
    }

    @Override
    public void processElement(Item value,
                               ReadOnlyContext ctx,
                               Collector<String> out) throws Exception {

        final MapState<String, List<Item>> state = getRuntimeContext().getMapState(mapStateDesc);
        final Shape shape = value.getShape();
    
        for (Map.Entry<String, Rule> entry :
                ctx.getBroadcastState(ruleStateDescriptor).immutableEntries()) {
            final String ruleName = entry.getKey();
            final Rule rule = entry.getValue();
    
            List<Item> stored = state.get(ruleName);
            if (stored == null) {
                stored = new ArrayList<>();
            }
    
            if (shape == rule.second && !stored.isEmpty()) {
                for (Item i : stored) {
                    out.collect("MATCH: " + i + " - " + value);
                }
                stored.clear();
            }
    
            // there is no else{} to cover if rule.first == rule.second
            if (shape.equals(rule.first)) {
                stored.add(value);
            }
    
            if (stored.isEmpty()) {
                state.remove(ruleName);
            } else {
                state.put(ruleName, stored);
            }
        }
    }
}
```

## Important Considerations
* There is no cross-task communication:\
태스크간 통신이 없기 때문에 `(Keyed)BroadcastProcessFunction` 에서 broadcast 스트림 처리쪽 함수만 broadcast state 를 수정할 수 있다. 또한 모든 태스크가 동일한 방식을 broadcast state 를 수정해야한다. 그렇지 않으면 서로 다른 태스크 인스턴스가 서로 다른 컨텐츠를 가질 수 있고, 이는 일관성 없는 결과로 이어진다.

* Order of events in Broadcast State may differ across tasks:\
broadcast state 쪽의 element 가 서로 다른 순서로 각 태스크에 도착할 수 있다. 따라서 state 업데이트는 broadcast element 가 들어오는 순서에 의존성이 있어서는 안된다.

* All tasks checkpoint their broadcast state:\
모든 태스크들이 동일한 broadcast state 를 가지고 있으며, 체크포인트시에 모든 태스크가 그들의 broadcast state 를 각자 저장한다. 모든 태스크가 동일한 파일을 읽어 복구하는 것을 피하기 위한 디자인이며, 단점은 parallelism 에 비례하여 checkpoint 된 state 크기가 커진다는 것이다.

* No RocksDB state backend:\
Broadcast state 는 런타임에 in-memory 로 관리되며 따라서 memory provisioning 이 되어야 한다. (?) 모든 operator state 에 대해서 동일하다..?\
아래 SO 참고
- https://stackoverflow.com/questions/62509773/flink-broadcast-state-rocksdb-state-backend?rq=1
- https://stackoverflow.com/questions/55213502/flink-broadcast-state-doesnt-save-to-rocksdb-when-checkpoint-happened


---

## 직접 작성해본 예제코드
```
package com.example.app

import com.fasterxml.jackson.databind.ObjectMapper
import org.apache.flink.api.common.state.MapStateDescriptor
import org.apache.flink.api.scala.createTypeInformation
import org.apache.flink.configuration.{Configuration, GlobalConfiguration}
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment
import org.apache.flink.streaming.api.functions.ProcessFunction
import org.apache.flink.streaming.api.functions.co.BroadcastProcessFunction
import org.apache.flink.streaming.api.functions.source.SourceFunction
import org.apache.flink.streaming.api.scala.OutputTag
import org.apache.flink.streaming.connectors.netty.HttpReceiverSource
import org.apache.flink.util.Collector

import scala.util.{Failure, Success, Try}

case class FilterRule(filter: Int)

case class PrefixRule(prefix: String)

object BroadcastStateApp {

  def main(args: Array[String]) {
    val flinkConf = new Configuration()
    val env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(GlobalConfiguration.loadConfiguration(flinkConf))

    env.setParallelism(3)

    val filterTag: OutputTag[FilterRule] = OutputTag[FilterRule]("filterTag")
    val prefixTag: OutputTag[PrefixRule] = OutputTag[PrefixRule]("prefixTag")

    val source = env.addSource(new SourceFunction[Int]{
      override def run(ctx: SourceFunction.SourceContext[Int]): Unit = {
        var counter = 0
        while (true) {
          ctx.collect(counter)
          counter += 1
          Thread.sleep(1000L)
        }
      }
      override def cancel(): Unit = {}
    })
      .name("Counter")
      .disableChaining()

    val defaultStream = env.fromElements(
      """
        |{
        |    "filter": 5,
        |    "prefix": "xxxx"
        |}
        |""".stripMargin)
      .name("Default Config")
    val httpStream = env.addSource(new HttpReceiverSource(10000, "config"))
      .setParallelism(1)
      .name("Http Config")

    val configStream = httpStream.union(defaultStream)
      .process(new ProcessFunction[String, Int]{
        val om = new ObjectMapper

        override def processElement(value: String, ctx: ProcessFunction[String, Int]#Context, out: Collector[Int]): Unit = {
          Try {
            val o = om.readTree(value)
            if (o.has("filter")) {
              ctx.output(filterTag, FilterRule(o.get("filter").asInt()))
            }
            if (o.has("prefix")) {
              ctx.output(prefixTag, PrefixRule(o.get("prefix").asText()))
            }
          } match {
            case Success(_) => ()
            case Failure(exception) => exception.printStackTrace()
          }
        }
      })
      .setParallelism(1)
      .name("Process Config")

    val filterRuleStateDescriptor = new MapStateDescriptor(
      "filterRulesBroadcastState",
      createTypeInformation[String],
      createTypeInformation[FilterRule]
    )

    val filterConfigStream = configStream
      .getSideOutput(filterTag)
      .broadcast(filterRuleStateDescriptor)

    source
      .connect(filterConfigStream)
      .process(new BroadcastProcessFunction[Int, FilterRule, Int] {
        override def processElement(value: Int, ctx: BroadcastProcessFunction[Int, FilterRule, Int]#ReadOnlyContext, out: Collector[Int]): Unit = {
          val filter = Try(ctx.getBroadcastState(filterRuleStateDescriptor).get("filter")) match {
            case Success(null) => 1
            case Success(v) => v.filter
            case Failure(_) => 1
          }

          if (value % filter == 0) {
            out.collect(value)
          }
        }

        override def processBroadcastElement(value: FilterRule, ctx: BroadcastProcessFunction[Int, FilterRule, Int]#Context, out: Collector[Int]): Unit = {
          ctx.getBroadcastState(filterRuleStateDescriptor).put("filter", value)
        }
      })
      .name("Filter")
      .setParallelism(3)
      .print()


    val prefixRuleStateDescriptor = new MapStateDescriptor(
      "prefixRulesBroadcastState",
      createTypeInformation[String],
      createTypeInformation[PrefixRule]
    )

    val prefixConfigStream = configStream
      .getSideOutput(prefixTag)
      .broadcast(prefixRuleStateDescriptor)

    source
      .connect(prefixConfigStream)
      .process(new BroadcastProcessFunction[Int, PrefixRule, String] {
        override def processElement(value: Int, ctx: BroadcastProcessFunction[Int, PrefixRule, String]#ReadOnlyContext, out: Collector[String]): Unit = {
          val prefix: String = Try(ctx.getBroadcastState(prefixRuleStateDescriptor).get("prefix")) match {
            case Success(null) => "default"
            case Success(v) => v.prefix
            case Failure(_) => "default"
          }

          out.collect(s"$prefix-$value")
        }

        override def processBroadcastElement(value: PrefixRule, ctx: BroadcastProcessFunction[Int, PrefixRule, String]#Context, out: Collector[String]): Unit = {
          ctx.getBroadcastState(prefixRuleStateDescriptor).put("prefix", value)

        }
      })
      .name("Prefix")
      .setParallelism(3)
      .print()

    env.execute()
  }
}
```
