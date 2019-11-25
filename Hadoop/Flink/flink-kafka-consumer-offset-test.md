# Flink Kafka Consumer Offset Test (Checkpointing disabled)

Flink 의 Kafka consumer 를 checkpointing 없이 사용할 때 유실되는 메시지가 있는지에 대한 테스트

* 결론 먼저 말하면
```
카프카 토픽을 소스로 하는 flink job이 async 오퍼레이터를 갖거나 처리 시간이 오래 걸릴 경우
job 재시작시 가장 최근 offset 부터 데이터를 읽어오면 유실되는 메시지가 생길 수 있다.
```

* KafkaProducerTestJob.kt
  * 10초마다 한 번씩 test-topic 카프카 토픽으로 메시지를 전송
  * 메시지 내용은 1부터 시작해서 증가하는 숫자

* KafkaConsumerTestJob.kt
  * test-topic 카프카 토픽으로부터 메시지를 읽어와서 60초씩 기다렸다가 로그로 출력하는 싱크로 메시지를 보내주는 job
  * 카프카 컨슈머의 offset 커밋 주기는 기본값인 30초

## KafkaProducerTestJob.kt

```kotlin
import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.api.common.time.Time
import org.apache.flink.api.common.typeinfo.TypeInformation
import org.apache.flink.api.java.typeutils.ResultTypeQueryable
import org.apache.flink.configuration.Configuration
import org.apache.flink.configuration.GlobalConfiguration
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment
import org.apache.flink.streaming.api.functions.source.RichSourceFunction
import org.apache.flink.streaming.api.functions.source.SourceFunction
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer
import org.apache.flink.streaming.connectors.kafka.partitioner.FlinkKafkaPartitioner
import org.apache.flink.streaming.connectors.kafka.partitioner.FlinkRoundRobinPartitioner
import org.apache.kafka.clients.producer.ProducerConfig
import java.lang.reflect.ParameterizedType
import java.util.*

object KafkaProducerTestJob {

    class TimerSource<T>(private val interval: Time, private val values: List<T>) :
        ResultTypeQueryable<T>, RichSourceFunction<T>() {
        private var iterator: Iterator<T>? = null
        private var expireTime = System.currentTimeMillis()

        override fun getProducedType(): TypeInformation<T> {
            val type = (javaClass.genericSuperclass as ParameterizedType).actualTypeArguments[0]
            val clazz: Class<T>? = type.javaClass as? Class<T>
            return TypeInformation.of(clazz)
        }

        override fun open(parameters: Configuration?) {
            super.open(parameters)
            iterator = values.iterator()
        }

        override fun run(ctx: SourceFunction.SourceContext<T>?) {
            iterator?.run {
                while (this.hasNext()) {
                    while (expireTime > System.currentTimeMillis()) {
                        Thread.sleep(1)
                    }
                    ctx?.collect(this.next())
                    expireTime = System.currentTimeMillis() + interval.toMilliseconds()
                }
            }
            ctx?.close()
        }
        override fun cancel() {}
    }

    fun makeKafkaStringProducer(bootstrapServers: String, topic: String, useRoundRobinPartitioner: Boolean = false): FlinkKafkaProducer<String> {
        val props = Properties()
        props.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers)
        props.setProperty(ProducerConfig.RETRIES_CONFIG, "10")
        props.setProperty(ProducerConfig.RETRY_BACKOFF_MS_CONFIG, "1000")
        props.setProperty(ProducerConfig.REQUEST_TIMEOUT_MS_CONFIG, "300000")

        val partitioner: FlinkKafkaPartitioner<String> = FlinkRoundRobinPartitioner()

        if (useRoundRobinPartitioner)
            return FlinkKafkaProducer(topic, SimpleStringSchema(), props, Optional.of(partitioner))
        else
            return FlinkKafkaProducer(topic, SimpleStringSchema(), props)
    }

    fun getStreamExecutionEnvironment(port: Int? = null): StreamExecutionEnvironment {
        val conf = GlobalConfiguration.loadConfiguration()
        port?.let { conf.setInteger("rest.port", port) }
        return StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(conf)
    }
}

/**
 * 로컬에서 테스트시 VM options 에 아래 옵션처럼 log4j property 파일 설정 추가하고 실행
 * -Dlog4j.configuration=log4j-console.properties
 */
fun main() {
    val env = KafkaProducerTestJob.getStreamExecutionEnvironment(8082)
    env.addSource(
            KafkaProducerTestJob.TimerSource<String>(
                Time.seconds(10),
                (1..10000).map { it.toString() }.toList()
            )
        )
        .addSink(KafkaProducerTestJob.makeKafkaStringProducer(
            "kafka.bootstrap.server.com:9092",
            "test-topic",
            true
        ))

    env.execute("KafkaProducerTestJob")
}
```

## KafkaConsumerTestJob.kt

```kotlin

import org.apache.flink.api.common.serialization.SimpleStringSchema
import org.apache.flink.api.common.time.Time
import org.apache.flink.configuration.Configuration
import org.apache.flink.configuration.GlobalConfiguration
import org.apache.flink.streaming.api.datastream.AsyncDataStream
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment
import org.apache.flink.streaming.api.functions.async.ResultFuture
import org.apache.flink.streaming.api.functions.async.RichAsyncFunction
import org.apache.flink.streaming.api.functions.sink.RichSinkFunction
import org.apache.flink.streaming.api.functions.sink.SinkFunction
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer
import org.apache.kafka.clients.consumer.ConsumerConfig
import org.slf4j.Logger
import org.slf4j.LoggerFactory
import java.util.*
import java.util.concurrent.CompletableFuture
import java.util.concurrent.TimeUnit


object KafkaConsumerTestJob {
    class PrintSink<T>: RichSinkFunction<T>() {
        private var log: Logger? = null

        override fun open(parameters: Configuration?) {
            super.open(parameters)
            log = LoggerFactory.getLogger(this.javaClass)
        }
        override fun invoke(value: T, context: SinkFunction.Context<*>?) {
            super.invoke(value, context)
            log?.info(value.toString())
        }
    }

    class WaitAsyncFunction<T>(private val time: Time): RichAsyncFunction<T, T>() {
        private var log: Logger? = null

        override fun open(parameters: Configuration?) {
            super.open(parameters)
            log = LoggerFactory.getLogger(this.javaClass)
        }
        override fun asyncInvoke(input: T, resultFuture: ResultFuture<T>?) {
            CompletableFuture.supplyAsync {
                log?.info("${input.toString()} sleeping in ${Thread.currentThread().name}")
                Thread.sleep(time.toMilliseconds())
                input
            }.thenAccept{ resultFuture?.complete(listOf(it)) }
        }
    }

    fun makeKafkaStringConsumer(bootstrapServers: String, groupId: String, topic: String, startFromLatest: Boolean = false): FlinkKafkaConsumer<String> {
        val props = Properties()
        props.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers)
        props.setProperty(ConsumerConfig.GROUP_ID_CONFIG, groupId)
        props.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "latest")
        props.setProperty(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "30000")
        props.setProperty(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true")

        val consumer = FlinkKafkaConsumer(topic, SimpleStringSchema(), props)
        if (startFromLatest)
            consumer.setStartFromLatest()

        return consumer
    }

    fun getStreamExecutionEnvironment(port: Int? = null): StreamExecutionEnvironment {
        val conf = GlobalConfiguration.loadConfiguration()
        port?.let { conf.setInteger("rest.port", port) }
        return StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(conf)
    }
}

/**
 * 로컬에서 테스트시 VM options 에 아래 옵션처럼 log4j property 파일 설정 추가하고 실행
 * -Dlog4j.configuration=log4j-console.properties
 */
fun main() {
    System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "10000")

    val env = KafkaConsumerTestJob.getStreamExecutionEnvironment()

    val instream = env
        .addSource(
            KafkaConsumerTestJob.makeKafkaStringConsumer(
                "kafka.bootstrap.server.com:9092",
                "test-topic-consumer-group",
                "test-topic",
                false
            )
        )

    val asyncStream = AsyncDataStream.unorderedWait(
        instream,
        KafkaConsumerTestJob.WaitAsyncFunction<String>(Time.seconds(60)),
        1,
        TimeUnit.HOURS,
        10000000
        ).disableChaining()

    asyncStream.addSink(KafkaConsumerTestJob.PrintSink<String>())

    env.execute("KafkaConsumerTestJob")
}
```