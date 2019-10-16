# Jackson 관련 annotations

* Jackson 은 java 기반의 JSON 라이브러리이다.
* 자주 사용하는 jackson annotation 들을 모아보려고 한다.
* 예제는 kotlin 으로 작성했다

## Sample Json

```json
{
    "firstName":"John",
    "lastName":"Smith",
    "age":25,
    "address":{
        "streetAddress":"21 2nd Street",
        "city":"New York",
        "state":"NY",
        "postalCode":"10021"
    },
    "phoneNumber":[
        {
            "type":"home",
            "number":"212 555-1234"
        },
        {
            "type":"fax",
            "number":"646 555-4567"
        }
    ]
}
```

## 정상적으로 파싱하는 예

```kotlin
    data class Person(
        var age: Int?,
        var firstName: String?,
        var lastName: String?,
        var address: Any?,
        var phoneNumber: Any?
    )

    @Test
    fun jsonParseTest() {
        val om = jacksonObjectMapper()
        val p = om.readValue(person, Person::class.java)
        println(om.writeValueAsString(p))
    }
```

## JsonProperty
* 클래스의 변수 이름과 json 의 필드 이름이 다를 때 명시적으로 지정하기 위해 사용한다.
* 아래 예제에서는 `fn`, `ln` 변수에 `firstName`, `lastName` json 필드 내용을 읽어온다.

```kotlin
    data class Person(
        var age: Int?,
        @JsonProperty("firstName")
        var fn: String?,
        @JsonProperty("lastName")
        var ln: String?,
        var address: Any?,
        var phoneNumber: Any?
    )
```

## JsonIgnoreProperties
* class 에 다는 annotation 이고, 무시할 프로퍼티들을 지정하거나 선언되지 않은 프로퍼티들은 무시되도록 설정할 때 사용한다.
* 아래 예제는 `address`, `phoneNumber` 를 선언하지 않아 `UnrecognizedPropertyException` 이 발생한다.

```kotlin
    data class Person(
        var age: Int?,
        var firstName: String?,
        var lastName: String?
    )
```
* 아래처럼 수정하면 예외가 발생하지 않고, 지정된 변수만 파싱된다.

```kotlin
    @JsonIgnoreProperties("address", "phoneNumber")
    data class Person(
        var age: Int?,
        var firstName: String?,
        var lastName: String?
    )

    // 또는
    @JsonIgnoreProperties(ignoreUnknown = true)
    data class Person(
        var age: Int?,
        var firstName: String?,
        var lastName: String?
    )
```

## JsonInclude
* serialize 할 때, 값이 "non-value" (null 이나 비어있는 값) 인 필드가 포함되면 안되는 경우 사용한다.
* 프로퍼티나 클래스에 붙일 수 있다.
* 예제 json 과 data class 가 아래와 같을 때, 역직렬화 > 직렬화 단계를 거치면 `address`, `phoneNumber` 필드가 사라진다.

```json
{
    "firstName":"John",
    "lastName":"Smith",
    "age":25,
    "address":null,
    "phoneNumber":null
}
```

```kotlin
    @JsonInclude(JsonInclude.Include.NON_NULL)
    data class Person3(
        var age: Int?,
        var firstName: String?,
        var lastName: String?,
        var address: Any?,
        var phoneNumber: Any?
    )
```

* annotation 의 value 값으로 넣을 수 있는 타입은 `NON_NULL` 말고도 여러가지가 있다.
    * `ALWAYS` - 선언된 모든 필드를 무시하지 않음
    * `NON_NULL` - 값이 null 인 필드는 무시
    * `NON_EMPTY` - 값이 null 이거나 비어 있다고 판단되는 필드는 무시