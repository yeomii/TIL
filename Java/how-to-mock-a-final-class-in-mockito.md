# Mockito 에서 final 클래스를 mock 으로 만들어 주기

* Mockito 를 사용해서 mock 객체를 만들다 보면 아래와 같은 에러가 발생하는 경우가 있다.
```
Mockito cannot mock/spy because :
 - final class
```

* Mockito 1.x 대에서는 위와 같이 final 클래스에 대한 mock 객체를 만드는 것이 불가능했지만 2.x 대에서는 `mockito-inline` 의존성을 추가하면 가능하다.

```
// gradle dependency

    testImplementation("org.mockito:mockito-core:$mockitoVersion")
    testImplementation("org.mockito:mockito-inline:$mockitoVersion")

    ...
```

* https://stackoverflow.com/questions/14292863/how-to-mock-a-final-class-with-mockito/40018295#40018295
