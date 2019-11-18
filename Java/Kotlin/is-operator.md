# `is` operator

* https://kotlinlang.org/docs/reference/typecasts.html

* kotlin 에서 인스턴스의 타입체크를 할 때 `is` 또는 `!is` 키워드를 주로 사용한다.
* java 의 `instanceof` 와 같은 기능을 하며, `is` 의 경우 추가로 타입 캐스팅도 해준다.

```java
if (x instanceof Integer) {
    System.out.println(((Integer) x) + 1);
} else if (x instanceof String) {
    System.out.println(((String) x).length() + 1);
}
...
```

```kotlin
when (x) {
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is IntArray -> print(x.sum())
}
```

* `is` 의 경우 클래스를 비교할 때 하위 클래스도 같은 클래스로 인식하기 때문에 정확한 타입을 비교하려면 클래스를 직접 비교해야 한다.
```kotlin
open class Animal(val name: String)
class Bee: Animal("Bee")

@Test
fun typeCheckTest() {
    val bee = Bee()
    println("bee is an animal : ${bee is Animal}") 
    // bee is an animal : true

    println("bee's class is Animal : ${bee.javaClass == Animal::class.java}")
    // bee's class is Animal : false
}
```