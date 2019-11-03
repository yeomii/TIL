# 함수형 코틀린

* http://www.yes24.com/Product/goods/69086858
* https://play.kotlinlang.org (같이 사용하자)

---

## 01 코틀린: 데이터타입, 오브젝트, 클래스
* 코틀린은 일부 함수형 기능이 포함된 OOP 언어
* 언어 컨스트럭트 (language construct)
    * 허용되는 구문을 말하는 멋진 방법이라고 함

### 언어 컨스트럭트
* 클래스
    * 코틀린의 기본 타입
    * 인스턴스에 상태, 동작, 타입을 제공하는 템플릿
    * property (속성) - 클래스 상태는 속성으로 표시됨. `has-a` 관계.
    * 메소드 - 클래스의 행동은 메소드로 정의함. 기술적으로 메소드는 멤버함수.

* 상속
    * `is-a` 관계.
    * 코틀린에서는 `open` 변경자를 사용하여 선언한 클래스만 확장 가능.
    * 일반화 (generalization) - 일반적인 행동과 상태를 부모 클래스로 옮기는 과정
    * 오버라이드 - 메소드의 정의를 서브클래스에서 변경하는 것
    * 전문화 (specialization) - 계층구조에서 클래스를 확장하고 동작을 오버라이드 하는 프로세스

* 추상 클래스
    * 확장을 위해서만 디자인 된 클래스
    * `abstract` 와 `open` 의 차이는 `open` 만이 인스턴스화 할 수 있게 한다는 것
    * object 표현식 - 타입을 확장하는 익명 클래스의 인스턴스를 정의
```kotlin
abstract class BakeryGood(val flavour: String) {
    fun eat(): String {
        return "Yummy, $flavour ${name()}"
    }
    abstract fun name(): String
}

val food: BakeryGood = object: BakeryGood("Raspberry") {
    override fun name(): String { return "Pie" }
}
```

* 인터페이스
    * 클래스는 동시에 두 클래스를 확장할 수 없지만 여러 인터페이스를 확장할 수는 있음.

* open / abstract / interface 를 각각 사용해야 하는 케이스
    * open class
        * 확장하고 인스턴스화 되어야 하는 클래스
    * abstract class
        * 인스턴스화 할 수 없는 클래스
        * 생성자가 필요
        * 초기화 로직이 있음
    * interface
        * 다중 상속 적용
        * 초기화 로직이 필요 없음

* 오브젝트
    * 자연스러운 싱글톤
    * 타입이 없는 오브젝트 표현식은 메소드 내부에서 로컬로 사용되거나 클래스 내에서 private 하게만 사용할 수 있다.
    * 오브젝트 선언 - 오브젝트에 이름을 부여
    * 컴패니언 오브젝트
        * 클래스 / 인터페이스 내에서 선언된 오브젝트는 컴패니언 오브젝트로 표시될 수 있음
```kotlin
class CupCake(val flavour: String) {
    fun name(): String { return "$flavour 컵케이크" }

    companion object { // 이름을 줄 수도 있음
        fun almond(): CupCake { return CupCake("아몬드") }
        fun cheese(): CupCake { return CupCake("치즈") }        
    }
}

val almondCupCake = CupCake.almond()
```

* 제네릭
    * 제네릭 프로그래밍 - 일반적인 문제를 해결하는 알고리즘 생성 (+ 데이터 구조) 에 중점을 둔 스타일 프로그래밍
    * 코틀린은 타입 파라미터를 사용해 제네릭 프로그래밍 지원

* 타입 앨리어스
    * 기존 타입의 이름을 정의하는 방식을 제공
```kotlin
typealias Oven = Machine<Bakeable>
```

* Nullable 타입
    * if 의 조건절에서 null 값인지 확인하면 if 블록 내에서 코틀린은 해당 값을 nullable 하지 않은 값으로 스마트 캐스팅을 해준다.
    * 안전 호출 - `?.` 연산자
    * 앨비스 연산자 - `?:` 식의 값이 null 이면 대체값으로 선언된 값을 반환
    * `!!` - null 일 경우 NPE 발생

* 코틀린 타입시스템
    * Any - 코틀린의 모든 타입은 Any 로 부터 확장. Any 와 Any? 는 다른 타입
    * 최소 공통 타입
        * 타입시스템으로 인해 어떤 타입이 반환될 지 모를 때 코틀린은 최소 공통 타입이 아닌 Any 타입으로 판단한다.
    * Unit - void 대신 사용. 표현식이 반환보다는 부수 효과를 위해 호출됨을 의미
    * Nothing 
        * 코틀린 계층 전체의 맨 아래에 있는 타입
        * Nothing 은 Nothing? 을 포함한 모든 코틀린 타입을 확장
        * 실행될 수 없는 표현식을 나타낸다.
        * Nothing? 은 null 값의 타입

* 기타 타입
    *  데이터 클래스
        * 캐노니컬 메소드 (equals, hashCode, toString)
        * copy() 메소드
        * destructuring 메소드 (component1() 등..)
    * 주석 (annotations)
        * `annotation class Tasty` 와 같이 사용
        * 코드에 메타정보를 첨부하는 방법
        * nullable 해서는 안됨
        * 런타임시 annotation 값을 쿼리하려면 reflection API 를 사용해야 함 (kotlinreflect.jar 의존성)
    * 열거형 (enum)
        * `enum class Flour { WHEAT, CORN, CASSAVA }`
        * 인터페이스 확장 가능
        * 추상메소드를 가질 수도 있음
        * when 구문과 함깨 사용할 때 모든 case 가 커버되지 않으면 컴파일 에러가 발생할 수 있음

---

## 02 함수형 프로그래밍 시작

### 함수형 프로그래밍이란?
* 하나의 패러다임 (프로그램을 구성하는 스타일)
* 핵심은 표현식으로 데이터를 변환하는 것 (이상적으로는 부수 효과가 없어야 함)
* 같은 입력으로 호출된 함수는 항상 같은 값을 반환하도록 보장
* 이점
    * 코드는 읽기 쉽고 테스트하기 쉽다 - 함수가 외부 가변 상태에 의존하지 않기 때문
    * 상태와 부수 효과가 주의 깊게 계획된다
    * 동시성이 좀 더 안전해지며 더 자연스러워 진다

### 기본 개념
* 일급 함수 (fist-class function)
    * 함수를 다른 타입으로 취급하여 변수, 파라미터, 반환, 일반화 타입 등으로 사용할 수 있게 한다.
* 고차 함수 (high-order function)
    * 다른 함수를 파라미터로 사용하거나 반환하는 함수
* 코틀린은 일급 함수와 고차 함수 컨셉을 모두 지원

### 예제
```kotlin
val capitalize = { str: String -> str.capitalize() }

// capitalize 와 동일
val capitalize2  = object: Function1<String, String> {
    override fun invoke(p1: String): String {
        return p1.capitalize()
    }
}
```
* 위 예제에서 `capitalize` 는 `(String) -> String` 타입 (syntactic sugar) 또는 `Function1<String, String>` 이다. 
* `Function1<P, R>` 은 코틀린 표준 라이브러리에 정의된 인터페이스로, `invoke(P) : R` 인 하나의 메소드다

```kotlin
fun <T> transform(t: T, fn: (T) -> T): T {
    return fn(t)
}

fun reverse(s: String): String { return str.reversed() }

val reversedStr = transform("abc", ::reverse)
val reversedStr2 = transform("abc", { s -> s.reversed() })
val reversedStr3 = transform("abc", { it.reversed() })
val reversedStr4 = transform("abc") { it.reversed() }
```
* 위 예제는 람다 함수를 다른 함수의 파라미터로 사용하는 예제이다.
* `::` 을 사용해 함수에 대한 참조를 전달할 수 있다
* 일반적으로 람다를 전달하고 (2), 암시적 파라미터를 사용하기도 한다. (3)
* 함수가 마지막 파라미터로 람다를 받으면, 람다는 괄호 밖으로 전달될 수 있다. (4)
    * 이 기능을 사용해 코틀린에서 DSL (domain specific language) 를 생성할 수 있다

```kotlin
fun unless(cond: Boolean, block: () -> Unit) {
    if (!cond) block()
}

val authenticated = false
unless(authenticated) { // if (!authenticated) 와 같음 
    print("인증되지 않은 사용자입니다")
}
```

### 순수함수
* 부수 효과나 메모리, IO 가 없다.
* 참조 투명도, 캐싱(메모이제이션) 등 다양한 속성을 가진다.
* 코틀린은 순수 함수를 작성하는 것은 가능하지만, 실행하지는 않을 것이다. 

### 재귀함수
* 실행을 멈추는 조건과 함께 스스로를 호출하는 함수다.
* `tailrec` 수정자를 통해 꼬리 재귀 함수의 스택 관리를 최적화 할 수 있다.
```kotlin
fun tailrecFactorial(n: Long): Long {
    tailrec fun go(n: Long, acc: Long): Long {
        return if ( n <= 0 ) acc else go(n - 1, n * acc)
    }
    return go(n, 1)
}
```

### 느긋한 계산법
* 일부 함수형 언어는 `lazy` 계산 모드를 제공한다.
* 코틀린은 기본적으로는 엄격한 평가를 사용하지만, 코틀린 표준 라이브러리와 `Delegate Properties` 를 언어 기능의 일부로 제공한다.

```kotlin
val i by lazy { 
    println("lazy evaluation") 
    1
}

println("before i")
println(i) // lazy evaluation 과 1 이 출력됨

// divide by 0 에 의한 ArithmeticException 이 발생하지 않음. 람다를 사용했기 때문에 lazy 하게 계산됨
println(listOf({2+1}, {1/0}).size) 
```
* 위 예제에서 `by` 예약어 뒤의 `lazy` 상위 함수는 `i` 에 처음 접근할 때 실행될 초기화 람다 함수를 받는다.
* 또 일부 상황에서는 노멀 람다 함수를 사용할 수 있다.

### 함수적 컬렉션
* 고차 함수를 통해 요소와 상호작용할 수 있는 방법을 제공하는 컬렉션
* filter, map, fold 등의 공통된 작업 제공
* 함수적 컬렉션은 순수 함수적 데이터 구조일 수도 있고 아닐 수도 있음
    * 순수 함수적 데이터 구조는 변경 불가능하며, 느긋한 계산법과 다른 기능 테크닉을 사용

---

## 03 불변성: 중요한 것
* 함수형 프로그래밍에서 가장 중요한 부분 중 한다

### 불변성(immutability) 이란?
* 무언가가 변할 수 없다는 것을 의미
    * immutable variable - 변경될 수 없는 변수
* 본질적으로 함수형 프로그래밍은 thread safe 하고, 불변성은 thread safe 하게 만드는 데 큰 역할을 한다.
* 실제로 불변성은 변경 금지에 대한 것 보다는 변경 처리에 대한 내용
    * 속성의 값을 직접 변경하는 대신 새 속성을 만들고 적용된 변경사항으로 값을 복사
* 코틀린에서 불변성 구현
    * 코틀린은 clojure, haskell 등과는 달리 불변성이 강제되는 순수 함수형 프로그래밍 언어가 아니다.
    * 코틀린에서는 불변성을 권장하며, 불변 변수 `val` 을 갖지만 상태의 진정한 깊은 불변성을 보장하는 언어 메커니즘은 없다.
    * 상태의 진정한 불변성은 속성이 언제나 같은 값을 반환할 것임을 의미



