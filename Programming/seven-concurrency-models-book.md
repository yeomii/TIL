# 7가지 동시성 모델

* http://m.hanbit.co.kr/store/books/book_view.html?p_code=B3745244799

# 01 서문

## 동시성과 병렬성
```
동시성은 여러 일을 한꺼번에 다루는 데 관한 것이다
병렬성은 여러 일을 한꺼번에 실행하는 데 관한 것이다
- Rob Pike
```

* 동시성과 병렬성이 혼동되는 이유는 정통적으로 사용하는 스레드와 잠금장치는 병렬성을 직접 지원하지 않기 때문이다.
* 이러한 스레드와 잠금장치를 이용해서 멀티코어를 활용하는 유일한 방법은 동시적인 프로그램을 작성해 병렬로 동작하는 하드웨어에서 실행하는 것이다.

## 병렬 아키텍처
* 비트 수준 병렬성
* 명령어 수준 병렬성
    * cpu 파이프라이닝, 비순차 실행, 추측 실행
* 데이터 병렬성 
    * SIMD
* 태스크 수준 병렬성
    * 메모리 모델
    * 공유 메모리 / 분산 메모리

## 일곱가지 모델
* 스레드와 잠금장치
    * 문제점이 많지만 다양한 기술의 기반이 되는 기본 기술
* 함수형 프로그래밍
    * 스레드 안전을 보장하여 병렬처리가 쉽다
* 클로저 방식 - 아이덴티티와 상태 분리하기
    * 클로저는 명령형과 함수형 프로그래밍을 효과적으로 결합
* 액터
    * 범용의 동시성 프로그래밍 모델
    * 공유 메모리와 분산 메모리 아키텍처 양측에서 활용가능
    * 지리적 분산을 적극 지원하고, 장애 허용과 탄력성에 대한 지원이 강함
* 순차 프로세스 통신 (Communicating sequential processes)
    * 액터 모델과 공통점이 많아보이지만 전혀 다른 특징을 가짐
    * 커뮤니케이션을 위해 액터같은 객체가 아닌 채널이라는 개념을 사용
* 데이터 병렬성
    * GPU 와 같은 경우
* 람다 아키텍처
    * 맵리듀스와 스트리밍 프로세스의 장점을 결합, 다양한 빅데이터 문제 해결

## 알맞은 모델을 고를 때 고려할 점
* 이 모델은 동시성 문제, 병렬성 문제, 혹은 두 개를 모두 해결하는 데 적합한가?
* 이 모델은 어떤 병렬 아키텍처를 타깃으로 삼고 있는가?
* 이 모델은 탄력성을 갖춘 코드, 혹은 지리적으로 분산된 코드를 작성할 때 필요한 도구를 제공하는가?

# 02 스레드와 잠금장치

* 스레드와 잠금장치는 하드웨어 동작방식을 거의 그대로 옮긴것과 비슷하다.
* 장점
    * 단순해서 대부분의 언어가 이 기능을 제공
* 단점
    * 너무 단순해서 안전한 코드를 작성하는데 도움을 주지도 않고, 유지보수하기도 어렵다

## 상호 배제와 메모리 모델
### 스레드 
- 자바에서 사용하는 동시성의 가장 기본 단위, 하나의 논리적 흐름
- 공유 메모리를 이용해서 다른 스레드와 의사소통함
- 멀티스레드 코드가 잘못될 수 있는 경우
    - 경쟁조건 / 메모리 가시성 / 데드락

### 스레드 생성 기본 예제
```kotlin
fun main(args: Array<String>) {
    val myThread = Thread{
        println("${Thread.currentThread().name} - hello from new thread")
    }

    myThread.start()
    Thread.yield()
    println("${Thread.currentThread().name} - hello from main")
    myThread.join()
}
```
* `join` 메서드는 해당 스레드가 동작을 멈출때까지 기다린다.
* `Thread.yield` 는 현재 실행 중인 스레드가 사용 중인 프로세서를 양보할 용의가 있음을 스케줄러에 알려주는 힌트이다.
    * 때문에 위 코드는 실행 순서가 항상 보장되지 않는다.
    * `Thread.yield` 를 주석처리 해도 실행 순서가 보장되지는 않는다.

## 잠금장치가 없이 공유자원에 접근하는 예제
```kotlin
class Counting {
    var count: Int = 0
    fun increment() { count++ }
}

class CountingThread(private val counting: Counting): Thread() {
    override fun run() {
        for (i in 1..10000)
            counting.increment()
    }
}

fun main(args: Array<String>) {
    val counting = Counting()
    val t1 = CountingThread(counting)
    val t2 = CountingThread(counting)

    t1.start(); t2.start()
    t1.join(); t2.join()
    println(counting.count)
}
```
* 위 코드 실행시 마지막 count 는 20000 보다 작은 숫자가 된다.
* `count++` 이라는 코드는 바이트 코드로 보면 `읽기-수정하기-쓰기` 패턴과 같다
* 위 상황에 대한 해법은 `count` 에 대한 접근을 동기화 하는 것이다
    * 자바 객체에 포함되어있는 내제된 잠금장치 (intrinsic lock) 사용
        * mutex, monitor, critical section 등으로도 지칭됨
```kotlin
class Counting {
    var count: Int = 0
    @Synchronized fun increment() { count++ }
}
```
* synchronized 함수 호출시 해당 객체가 가지고 있는 잠금장치를 먼저 획득해야 한다.
* kotlin 에는 synchronized 키워드가 없고, @Synchronized 어노테이션을 사용할 수 있다.
* java 로 하면 `public synchronized void increment() { count++ }`

## 공유 메모리 모델에서 최적화에 의해 생기는 문제
```kotlin
fun main(args: Array<String>) {
    var isReady = false
    var answer = 0
    val t1 = Thread { answer = 42; isReady = true }
    val t2 = Thread {
        if (isReady)
            println("The answer is $answer")
        else
            println("I don't know the answer")
    }
    t1.start(); t2.start()
    t1.join(); t2.join()
}
```
* 위 코드 실행시 아래와 같은 이상한 출력을 얻을 수도 있다 (확인은 안됨)
```
The answer is 0
```
* 이 이상한 응답은 여러가지 원인으로 발생할 가능성이 있다.
    * 컴파일러는 코드가 실행되는 순서를 바꿈으로써 정적 최적화를 수행할 수 있다.
    * jvm 은 코드가 실행되는 순서를 바꿈으로써 동적 최적화를 수행할 수 있다.
    * 코드를 실행하는 하드웨어도 코드의 순서를 바꾸는 것이 가능하다.
* 공유 메모리를 사용하는 병렬 컴퓨터는 이러한 최적화가 필요하므로 이러한 문제를 올바르게 다루기 위해 자바 메모리 모델을 잘 알아야 한다.

### 자바 메모리 모델
* https://docs.oracle.com/javase/specs/jls/se11/html/jls-17.html
* 한 스레드가 메모리에 가한 변화가 다른 메모리에 보이는 경우를 정의
* 읽는 쓰레드와 쓰는 쓰레드가 동기화되지 않으면 그러한 가시성이 보장되지 않는다.

## 식사하는 철학자 - 데드락
```kotlin
import kotlin.random.Random

data class Chopstick(val i: Int)
class Philosopher(val i: Int, private val left: Chopstick, private val right: Chopstick): Thread() {
    private val random = Random(System.currentTimeMillis())
    override fun run() {
        while (true) {
            sleep(random.nextLong(1))
            println("Philosopher $i starts to try holding chopsticks")
            synchronized(left) {
                synchronized(right) {
                    println("Philosopher $i got chopsticks of both sides")
                    sleep(random.nextLong(1))
                    println("Philosopher $i finished dining")
                }
            }
        }
    }
}

fun main(args: Array<String>) {
    val n = 5
    val chopsticks = (1..n).map { Chopstick(it) }
    val philosophers = (1..n).map {
        Philosopher(it, chopsticks[it - 1], chopsticks[it % n])
    }
    philosophers.forEach { it.start() }
    philosophers.forEach { it.join() }
}
```
* 위 코드는 데드락 문제의 전통적인 예인 식사하는 철학자를 구현한 코드이다
* 모든 철학자가 동시에 젓가락을 들어올리기 시작한다면 모두 자기 왼쪽에 있는 젓가락을 들어올린 상태로 멈추게 된다.
* 이런 데드락을 피하려면 잠금장치를 요청할 때 항상 미리 정해진 공통의 순서를 따르면 된다.
    * 예를 들어 양쪽 젓가락 중 항상 홀수번째 젓가락을 먼저 들어올린다
    * 잠금장치에 대한 id 값은 고유하고 순서가 매겨져 있으면 id 로 사용하기 충분하다.

## 외부메서드의 위험
* 잠금장치를 보유하고 있는 상태에서 외부메서드를 호출하게 될 경우, 해당 외부메서드도 다른 잠금장치를 요청할 수도 있기 때문에 데드락의 위험을 초래할 수 있다.

## 추가 레퍼런스
* [william pugh - java memory model](http://www.cs.umd.edu/~pugh/java/memoryModel/)
* [JSR 133 (java memory model) FAQ](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr-133-faq.html)
* [double checked locking (anti-pattern)](https://en.wikipedia.org/wiki/Double-checked_locking)

## 내재된 잠금장치의 한계
* 내재된 잠금장치를 얻으려고 하다 블로킹 상태가 된 스레드를 원상복귀 시킬 수 없다
* 내재된 잠금장치를 얻으려고 노력하는 시간을 강제로 중단하는 타임아웃 기능이 없다
* 내재된 잠금장치를 얻는 방법은 `synchronized` 블록을 사용하는 한가지 방법 뿐이다

### 데드락 걸린 스레드 죽이기
```kotlin 
class UnInterruptable(val i: Int, val o1: Object, val o2: Object): Thread() {
    override fun run() {
        try {
            synchronized(o1) {
                sleep(1000)
                synchronized(o2) {
                }
            }
        } catch (e: InterruptedException) { println("thread $i interrupted") }
    }
}

fun main() {
    val o1 = Object(); val o2 = Object()
    val t1 = UnInterruptable(1, o1, o2)
    val t2 = UnInterruptable(2, o2, o1)
    t1.start(); t2.start()
    Thread.sleep(2000)
    t1.interrupt(); t2.interrupt()
    t1.join(); t2.join()
}
```
* 자바에서 스레드를 정상적으로 종료하는 방법은 `run` 메서드가 리턴하는 방법 뿐이다
* 따라서 스레드가 내재된 잠금장치 때문에 데드락이 걸린 경우 할 수 있는 일은 없다
* JVM 전체를 죽이는 것 외에 다른 방법이 없다

### ReentrantLock
* `synchronized` 보다 명시적인 잠금장치

