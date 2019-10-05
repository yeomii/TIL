# Flink Overview

## [Apache Flink](https://flink.apache.org/)
프레임워크이며 분산 처리를 위한 엔진으로, 데이터 스트림에 대한 연산을 수행할 수 있다. 일반적인 모든 클러스터 환경에서 동작할 수 있도록, 그리고 어떤 규모에서 수행하든 in-memory 연산 수준의 속도 성능을 가대할 수 있도록 디자인되었다. (Flink Architecture 문서)

* 기본적으로는 JVM 에서 동작하고, java, kotlin, scala, python 을 지원한다.
* 처리할 데이터의 형태에 따라 Streaming, Batch, Table API 를 사용할 수 있다.
* 개인적으로는 데이터 스트림 처리 코드를 깔끔하게 작성하기 쉬워서 Streaming API 위주로 사용하고 있다.

## [Streaming API](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/datastream_api.html)

### 예제 프로그램

* 아래 예제는 5초의 time window 동안 localhost:9999 소켓으로 들어온 단어들의 각 갯수를 계산하는 프로그램이다.
* 실행 전에 `nc -lk 9999` 명령어로 input stream 이 들어갈 소켓을 미리 열어줘야 한다.
* 코드는 flink-java, flink-streaming-java 에 의존성이 있고, flink 공식 예제를 kotlin 으로 작성해보았다.

```kotlin
package com.test.flink

import org.apache.flink.api.common.functions.FlatMapFunction
import org.apache.flink.api.java.tuple.Tuple2
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment
import org.apache.flink.streaming.api.functions.sink.SinkFunction
import org.apache.flink.streaming.api.windowing.time.Time
import org.apache.flink.util.Collector

fun main() {
    val env = StreamExecutionEnvironment
        .getExecutionEnvironment()

    env.socketTextStream("localhost", 9999)
        .flatMap(Splitter())
        .keyBy(0)
        .timeWindow(Time.seconds(5))
        .sum(1)
        .addSink(Printer())

    env.execute("Window Word Count Job")
}

class Splitter: FlatMapFunction<String, Tuple2<String, Int>> {
    override fun flatMap(value: String?, out: Collector<Tuple2<String, Int>>?) {
        value?.split(" ")?.forEach { out?.collect(Tuple2(it, 1)) }
    }
}

class Printer: SinkFunction<Tuple2<String, Int>> {
    override fun invoke(value: Tuple2<String, Int>?, context: SinkFunction.Context<*>?) {
        super.invoke(value, context)
        println(value)
    }
}
```

* 플링크 예제 프로그램은 위와 같이 세가지 단계로 구분할 수 있다.
    * StreamExecutionEnvironment 을 로드하는 부분
    * 가져온 실행환경에 데이터 소스 및 처리 스트림을 정의하는 부분
    * 정의한 실행환경을 실제로 실행시키는 부분
* 데이터 소스 및 처리 스트림을 정의하는 부분을 구현하여 원하는 데이터 처리 스트림을 만들수 있다. (예제에서는 Splitter 와 Printer 함수가 그러하다)
