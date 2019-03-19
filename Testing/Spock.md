# Spock

* spock 은 Groovy 언어로 작성된 java, groovy 기반 애플리케이션 테스팅 프레임워크이다.
* 테스트 코드를 간결하고 예쁘게 작성할 수 있는 것이 장점이다.
* [공식 문서](http://docs.spockframework.org/)

---

## 빠르게 시작하기
* [Spock Web Console](http://meetspock.appspot.com/)

## spock 테스트 코드 작성하기
java 기반 프로젝트에서 간단하게 리스트를 테스트해보는 코드를 작성해보겠다.


@ build.gradle
```gradle
apply plugin: 'groovy'
...
dependencies {
  // mandatory dependencies for using Spock
  compile "org.codehaus.groovy:groovy-all:2.4.15"
  testCompile "org.spockframework:spock-core:1.2-groovy-2.4"
}
...
```

@ src/test/groovy/com/test/ListTest.groovy
```groovy
package com.test

import spock.lang.Specification

class ListTest extends Specification {

    def addTest() {
        // test 를 위한 setup 코드를 작성하는 block
        given:
        def list = new ArrayList()
        def elem = 1

        // stimulus, 테스트를 할 method 를 호출하는 block
        when:
        list.add(elem)

        // response, 예상되는 결과를 검증하는 block
        then:
        list.size() == 1
        list.last() == elem
    }
}
```

## 주요 기능
### Fixture methods
* 테스트 환경을 셋업하고 정리하는데 책임이 있는 메소드들이다.
* 실행 순서에 따라 다음 네 가지 메소드들이 있다.
```groovy
def setupSpec() {} // 한번만 수행 - 첫 feature method 가 수행되기 전에
def setup() {} // 여러번 수행 - 각 feature method 들이 수행되기 전에
def cleanup() {} // 여러번 수행 - 각 feature method 들이 수행된 후에
def cleanupSpec() {} // 한번만 수행 - 마지막 feature method 가 수행된 후에
```

### Feature methods
* 실제로 테스트하고자 하는 내용이 들어가는 메소드이다.
* 네이밍 컨벤션은 string literal 로 작성하는 것이다.
```groovy
def "add one element to the list"() {
    // block 작성
}
```

### Blocks
* spock 이 제공하는 block 은 feature method 를 구성하는 개념적인 단계를 쉽게 작성할 수 있도록 해준다.
* 아래 여섯 가지의 블록이 제공된다. 
    * given
        * setup 코드가 들어간다.
        * optional
    * when 
        * stimulus
        * then 과 한 쌍으로, 테스트 할 함수를 호출하거나 하는 테스트 대상 코드가 들어가는 블록이다
    * then
        * response
        * when 과 한 쌍으로, when 블록을 실행시키고 나서 기대되는 결과를 확인하는 블록이다
    * expect
        * then 과 비슷하긴 하지만 condition 과 variable 정의만 담을 수 있다는 점에서 더 제한적인 블록이다.
        * 기대되는 결과가 하나의 expression 만으로 표현될 때 유용하다.
    * cleanup
        * 테스트시 사용한 리소스를 정리하는 블록이다
    * where
        * 항상 feature method 의 마지막에 위치한다
        * 그렇지만 feature method 가 실행되기 전에 평가된다
        * data-driven feature method 를 작성할 때 유용하게 쓰인다


## Spring 프로젝트에서 사용하기
* TODO