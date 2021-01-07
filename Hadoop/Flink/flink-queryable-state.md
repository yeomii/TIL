# Flink Queryable State

(Draft)

https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/stream/state/queryable_state.html

내용은 나중에 정리..

## Examples
* 예제 코드
```scala
package com.example.app

import com.fasterxml.jackson.databind.{JsonNode, ObjectMapper}
import org.apache.flink.api.common.functions.{FlatMapFunction, MapFunction, RichFlatMapFunction}
import org.apache.flink.api.common.state.{ValueState, ValueStateDescriptor}
import org.apache.flink.api.common.typeinfo.TypeInformation
import org.apache.flink.api.scala.createTypeInformation
import org.apache.flink.configuration.{Configuration, GlobalConfiguration}
import org.apache.flink.queryablestate.client.QueryableStateClient
import org.apache.flink.streaming.api.functions.source.{RichSourceFunction, SourceFunction}
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.streaming.connectors.netty.HttpReceiverSource
import org.apache.flink.util.Collector

import java.util.concurrent.CompletableFuture
import java.util.function.Consumer
import collection.JavaConverters._
import scala.util.{Failure, Success, Try}

class SimpleQueryableConfig {

}

object SimpleQueryableConfig {
  def main(args: Array[String]): Unit = {
    val config = new Configuration()
    config.setString("queryable-state.enable", "true")

    val env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(config)
    env.setParallelism(2)

    val source = env.addSource(new RichSourceFunction[Int]{
      var interval = 1000
      var counter = 0
      override def open(parameters: Configuration): Unit = {
        super.open(parameters)
      }

      override def run(ctx: SourceFunction.SourceContext[Int]): Unit = {
        while (true) {

          ctx.collect(counter)
          counter += 1
          Thread.sleep(interval)
        }
      }

      override def cancel(): Unit = {
      }
    })
      .disableChaining()

    source.print()

    val configStream = env.addSource(new HttpReceiverSource(10000, "config"))
      .setParallelism(1)
      .keyBy{x => "interval"}
      .flatMap(new RichFlatMapFunction[String, Int]{
        val om = new ObjectMapper
        var interval: ValueState[(String, Int)] = _
        override def flatMap(value: String, out: Collector[Int]): Unit = {
          Try {
            value.toInt
          } match {
            case Success(intValue) =>
              interval.update("interval", intValue)
              out.collect(intValue)
            case Failure(exception) => exception.printStackTrace()
          }
        }

        override def open(parameters: Configuration): Unit = {
          super.open(parameters)
          val intervalDesc = new ValueStateDescriptor("interval", createTypeInformation[(String, Int)])
          intervalDesc.setQueryable("interval")
          interval = getRuntimeContext.getState(intervalDesc)
        }
      })
      .setParallelism(1)
      .print()

    val job = env.executeAsync()

    Thread.sleep(10000)
    val client = new QueryableStateClient("localhost", 9069)
    while (true) {
      val intervalDesc = new ValueStateDescriptor("interval", createTypeInformation[(String, Int)])
      val f: CompletableFuture[ValueState[(String, Int)]] = client
        .getKvState(job.getJobID, "interval", "interval", createTypeInformation[String], intervalDesc)


      f.thenAccept(new Consumer[ValueState[(String, Int)]] {
        override def accept(t: ValueState[(String, Int)]): Unit = {
          println(t.value())
        }
      })
      Thread.sleep(1000)
    }
  }
}
```

* 추가 dependency
```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-queryable-state-runtime_2.11</artifactId>
            <version>1.10.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-queryable-state-client-java</artifactId>
            <version>1.10.0</version>
        </dependency>
    </dependencies>
```