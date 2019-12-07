# Java Serializable Object
* https://docs.oracle.com/javase/tutorial/jndi/objects/serial.html

## 직렬화 가능한 객체
* 객체 직렬화 
    - 객체의 상태를 byte stream 으로 바꾸는 것
    - 해당 byte stream 으로부터 기존 객체의 복제본을 복원해낼 수 있다.
* Java 객체가 `serializable` 하다는 것
    - 해당 클래스 또는 부모클래스가 `java.io.Serializable` 또는 `java.io.Externalizable` 인터페이스를 구현할 경우
* java 플랫폼은 직렬화 가능한 객체를 직렬화 하는 기본 방식을 정의한다.
    * java 클래스별로 이러한 기본 직렬화 함수를 override 하여 객체를 직렬화 하는 커스텀한 방식을 정의할 수 있다
    * 자세한 객체 직렬화 방식 설명은 링크 참고 [Object Serialization Specification](https://docs.oracle.com/javase/8/docs/technotes/guides/serialization/index.html)
* 객체를 직렬화 할때는 클래스 파일 자체에 대한 내용은 기록되지 않는다.

## 추가 reference 
* http://woowabros.github.io/experience/2017/10/17/java-serialize.html
* https://stackoverflow.com/questions/285793/what-is-a-serialversionuid-and-why-should-i-use-it