# 함수형 코틀린

* http://www.yes24.com/Product/goods/69086858

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


