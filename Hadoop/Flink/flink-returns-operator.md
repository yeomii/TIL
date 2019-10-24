# Flink Streaming API 의 returns operator

* flink 에서 streaming API 를 사용하여 job 을 구성할 때 아래와 같이 flatmap 등의 operator 에 generic 타입의 함수를 사용하면 런타임 에러가 발생한다.

```kotlin
package com.test.flink

import com.fasterxml.jackson.databind.ObjectMapper
import com.fasterxml.jackson.module.kotlin.jacksonObjectMapper
import org.apache.flink.api.common.functions.RichFlatMapFunction
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment
import org.apache.flink.util.Collector

fun main() {
    val env = StreamExecutionEnvironment
        .getExecutionEnvironment()

    env.fromElements(
        """{"userId":1,"id":1,"title":"delectus aut autem","completed":false}""",
        """{"userId":1,"id":2,"title":"quis ut nam facilis et officia qui","completed":false}""",
        """{"userId":1,"id":3,"title":"fugiat veniam minus","completed":false}""",
        """{"userId":1,"id":4,"title":"et porro tempora","completed":true}""",
        """{"userId":1,"id":5,"title":"laboriosam mollitia et enim quasi adipisci quia","completed":false}"""
        ).flatMap(JsonStringToObjectFunction(User::class.java))
        .print()

    env.execute("JsonStringToObjectJob")
}

data class User(var id: Int, var userId: Int, var title: String, var completed: Boolean)

class JsonStringToObjectFunction<T>(private val klass: Class<T>) : RichFlatMapFunction<String, T>() {
    private var om: ObjectMapper? = null
    override fun open(parameters: Configuration?) {
        super.open(parameters)
        om = jacksonObjectMapper()
    }

    override fun flatMap(value: String?, out: Collector<T>) {
        value?.let {s ->
            try {
                om?.readValue(s, klass)?.let { o -> out.collect(o) }
            } catch (e: Exception) {
                e.printStackTrace()
            }
        }
    }
}
```

* 에러 메시지는 다음과 같은데, 이유는 자바의 generic 은 type erasure 가 발생하기 때문에 flink 에서 타입추론을 자동으로 할 수 없기 때문이다.

```
Exception in thread "main" org.apache.flink.api.common.functions.InvalidTypesException: The return type of function 'main(TestJob.kt:20)' could not be determined automatically, due to type erasure. You can give type information hints by using the returns(...) method on the result of the transformation call, or by letting your function implement the 'ResultTypeQueryable' interface.
...
Caused by: org.apache.flink.api.common.functions.InvalidTypesException: Type of TypeVariable 'OUT' in 'class org.apache.flink.api.common.functions.RichFlatMapFunction' could not be determined. This is most likely a type erasure problem. The type extraction currently supports types with generic variables only in cases where all variables in the return type can be deduced from the input type(s). Otherwise the type has to be specified explicitly using type information.
```

* flink 에게 return type 의 힌트를 주려면 `returns` 오퍼레이터를 사용하거나, FlatMapFuntion 이 `ResultTypeQueryable` 인터페이스를 구현하도록 하면 된다.

* `returns` 오퍼레이터 사용
```kotlin
    env.fromElements(
        ...
        ).flatMap(JsonStringToObjectFunction(User::class.java)).returns(User::class.java)
        .print()
```

* `ResultTypeQueryable` 인터페이스 구현
```kotlin
class JsonStringToObjectFunction<T>(private val klass: Class<T>) : ResultTypeQueryable<T>, RichFlatMapFunction<String, T>() {
    ...
    override fun getProducedType(): TypeInformation<T> {
        return TypeInformation.of(klass)
    }
}
```
